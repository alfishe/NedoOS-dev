# System Architecture

*Up: [Documentation hub](../README.md) · Next: [Memory map](memory-map.md)*

This page gives the layered, bird’s-eye view of NedoOS. Each layer is then
expanded in dedicated pages: [memory](memory-map.md),
[processes](process-model.md), [I/O](io-and-pipes.md),
[interrupts](interrupt-and-timing.md), and the
[kernel internals](../03-kernel/kernel-structure.md).

## Layered view

```mermaid
flowchart TB
    subgraph HW["Hardware (ATM2 / ATM3 / ZX Evolution / PE26)"]
        Z80[Z80 CPU + IM1 interrupt]
        RAM[Banked RAM 1-3 MB<br/>16K pages]
        CRT[CRT controller 0xBD77<br/>EGA / MC / 6912 / text]
        KBD[Keyboard matrix or PS-2]
        MOU[Kempston mouse]
        FDC[VG93 FDC + TR-DOS ROM]
        IDE[IDE HDD]
        SDZ[Z-controller SD]
        NGSC[NeoGS / GS coprocessor]
        USBH[SL811 USB host]
        WIZ[WIZnet W5300]
        ESP[ESP8266 UART]
        RTC[Mr.Gluk RTC]
        AY[AY / TurboSound]
        COV[Covox DAC]
    end

    subgraph KERN["Kernel (system page at 0x0000, pgtrdosfs/pgfatfs/...)"]
        UK["userkrnl trampoline<br/>(copy per task at its 0x0000)"]
        SCHED[Scheduler + focus]
        BDOS[BDOS dispatcher<br/>sysbdos.asm]
        FS[Filesystems<br/>FatFS port + TR-DOS fs]
        DRV[Drivers<br/>fatfsdrv / trdosio / sl811 / w5300 / espnet / ps2drv / ngssddrv]
        NET[Network layer<br/>socket subfunctions]
        CON[Console + palette + gfx]
        INP[Input: keys, mouse, layouts]
        SND[Music + Covox player]
        CLK[Clock + timer]
    end

    subgraph SDKL["SDK (linked into each app)"]
        SYSH["sys_h.asm / sysdefs.asm<br/>OS_* macros"]
        STDIO["stdio.asm<br/>pipe-aware stdio"]
        STR["string.asm, file.asm, say.asm, ..."]
        NEDOLANGRT["NedoLang iofast runtime"]
    end

    subgraph APPS["Applications"]
        CMD["cmd shell"]
        TERM["term / netterm"]
        NV["nv file manager"]
        TOOLS["texted, scratch, view, ..."]
        NETAPPS["browser, telnet, ping, dmirc, ..."]
        GAMES["games + emulators"]
    end

    APPS --> SDKL --> UK --> BDOS
    BDOS --> SCHED
    BDOS --> FS --> DRV
    BDOS --> NET --> WIZ & ESP
    BDOS --> CON --> CRT
    BDOS --> INP --> KBD & MOU
    BDOS --> SND --> AY & COV
    BDOS --> CLK --> RTC
    DRV --> FDC & IDE & SDZ & NGSC & USBH
    SCHED --> Z80
    KERN -.-> RAM
```

## The three memory roles

Understanding NedoOS starts with realizing that the *same* physical address
space plays three roles simultaneously:

1. **User space** — each running task perceives `0x0100..0xffff` as its own
   machine (three freely bankable windows plus the trampoline window).
2. **Kernel space** — the system page mapped at `0x0000..0x3fff` (and scratch
   pages) holds the real kernel; it is *pulled in* through port `0xFD` writes
   performed by the trampoline before any OS call.
3. **Shared hardware state** — CRT controller, palette ports, FDC, IDE, SD,
   network and sound are reachable from the kernel only (user code gets them
   only through syscalls).

Details and exact page numbers: [memory map](memory-map.md).

## Control flow patterns

### OS call (user → kernel → user)

```mermaid
sequenceDiagram
    participant App as User app (any window)
    participant UK as userkrnl (0x0000 of task page)
    participant K as Kernel (system page)
    App->>App: ld c,CMD_... (setup args)
    App->>UK: CALLBDOS = ex af,af' / call 0x0005
    UK->>UK: ld a,fd_system / out (0xFD),a
    Note over UK: system page now visible at 0x0000
    UK->>K: jp callbdos (mutex, then dispatch)
    K->>K: execute CMD_ handler (BDOS pages)
    K->>App: return path restores task page via 0xFD
```

### Task switch on frame interrupt

```mermaid
sequenceDiagram
    participant HW as 50 Hz INT
    participant IH as 0x0038 entry (task page)
    participant K as Kernel sys_int
    HW->>IH: interrupt
    IH->>IH: push af/bc/de, out (0xFD),fd_system
    IH->>K: init_resident / sys_sysint
    K->>K: save full context into app safestack
    K->>K: music tick, palette/border of focus app
    K->>K: schedule() picks next active app
    K->>K: restore that app context and pages
    K-->>HW: ret with interrupts re-enabled
```

### Program launch

```mermaid
sequenceDiagram
    participant U as User (shell)
    participant C as cmd
    participant K as Kernel
    participant T as New task
    U->>C: type program.com args
    C->>K: open file, read pages, OS_NEWAPP
    K->>T: allocate 4 windows + userkrnl copy
    C->>T: write COMMANDLINE at 0x0080
    K->>T: set factive, schedule next frame
    T-->>C: (if blocking) WAITPID collects result
```

## Subsystem responsibilities

| Subsystem | Kernel files | Notes |
|---|---|---|
| Trampoline & restarts | `userkrnl.asm` | per-task copy; QUIT/BDOS/GETKEY/PRCHAR/SETPG |
| Scheduler, app table, focus | `syskrnl.asm` | `MAXAPPS=16` descriptors, `safestack` contexts |
| BDOS dispatcher | `sysbdos.asm`, table tail of `main.asm` | function-number → handler jump table |
| FAT stack | `fatfs4os/ff.c` + `fatfsdrv.asm` | C core cross-compiled (SDCC), drivers in asm |
| TR-DOS stack | `trdosfs.asm`, `trdosio.asm` | segmented-file support, R/I/A error UI |
| Block drivers | `fatfsdrv.asm`, `ngssddrv.asm`, `sl811.asm`, `xt_drv.asm` | IDE (2 schemes), SD (2 kinds), USB |
| Network | `w5300.asm`, `w5300ini.asm`, `espnet.asm`, `espnet_bss.asm` | selected by `INETDRV` |
| Input | `syskey1.asm`, `syskey2.asm`, `ps2drv.asm` | matrix + PS/2, RU/EN recoding |
| Console/gfx | parts of `sysbdos.asm`, `syskrnl.asm` | char out, colours, scroll, gfx modes, palettes |
| Sound | `main.asm` (music), Covox player | PT3 engine, PCM with page table |
| Idle & boot app | `idle.asm`, `main.asm` init part | mounts drives, logo, launches `term` |

## Key architectural decisions (and why)

1. **Trampoline instead of trap instructions.** Z80 lacks a cheap supervisor
   call; `rst` targets live in banked page zero, which each task owns. The
   trampoline plus the `0xFD` port write gives a 100% userspace-resolvable call
   path with kernel entry cost of ~3 instructions.
2. **Scheduling by frame interrupt, not by preemption.** Without an MMU or a
   free programmable timer, the video frame interrupt is the only dependable
   clock; the design accepts one-slice-per-frame fairness and asks long-running
   tasks to `YIELD`.
3. **CP/M + MSX-DOS layered API.** Reusing CP/M function numbers (0x05–0x22)
   and MSX-DOS handle calls (0x43–0x5e) buys source compatibility with a large
   retro software corpus, while 0xc6+ invents modern services (pages, pipes,
   gfx, net).
4. **FatFS as C island.** The FAT core is compiled C (`fatfs4os`) linked at
   `0x4000` of a dedicated kernel page — trading a rare language boundary for a
   battle-tested FS implementation.
5. **Everything per-task.** Current directory, DTA, palette, border, screen
   pages, stdin/stdout/stderr — all live in the app descriptor, enabling clean
   focus switching and independent consoles (`term` per shell).
6. **Compile-time hardware abstraction.** One source tree, many machines:
   `syssets.asm` + `atm=` conditionals; no runtime driver database — an
   appropriate trade for ROM-less disk boot and small size.

## What NedoOS deliberately is not

* Not Unix: no per-task address isolation, no permissions, one user.
* Not preemptive: a task that never yields still hogs CPU between interrupts
  of a frame — the kernel reclaims control only at the frame boundary.
* Not a multi-console GUI: focus moves between *visual tasks*, but there is
  exactly one visible screen and one input stream at a time.
* Not portable off the Z80: page sizes, ports and restarts are baked in.

## Next steps

* [Memory map](memory-map.md) — exact pages, windows, stacks, buffers.
* [Process model](process-model.md) — the app descriptor and scheduler in depth.
* [Kernel structure & boot](../03-kernel/kernel-structure.md) — how it all starts.
