# Glossary

*Prev: [Hardware platforms](hardware-platforms.md) · Up: [Documentation hub](../README.md)*

Terms are grouped by theme. Cross-references point into this documentation set
and into the actual sources.

## Core & memory

| Term | Meaning |
|---|---|
| **Z80** | 8-bit CPU of the ZX Spectrum family; 16-bit address space, little-endian |
| **Page** | 16 KiB unit of *physical* RAM (`страница` in the Russian docs) |
| **Window** | A 16K *address* interval `0x0000/0x4000/0x8000/0xC000` onto which pages are mapped (`окно`) |
| `pagexor` | Constant used to convert page numbers to hardware register values (`0x7f` on ATM2, `0xff` otherwise) |
| `pgsys`, `pgtrdosfs`, `pgfatfs`, `pgfatfs2`, `pgkillable` | Reserved system pages: kernel code, TR-DOS FS data, FAT data, FAT structures, kill-safe scratch (see [memory map](../02-architecture/memory-map.md)) |
| `pgscr0_0…pgscr1_1` | System screen pages (attributes/bitmap of the two hardware screens) |
| **userkrnl** | Per-task trampoline placed at page zero of every task ([`userkrnl.asm`](../../NedoOS/src/kernel/userkrnl.asm)) |
| `fd_system` / `fd_user` | Magic values (`0x57`/`0x47`) written to port `0xFD` to expose the system page or the task page at `0x0000..0x3FFF` |
| **TOPDOWNMEM** | Build flag that allocates system pages from the top of RAM instead of fixed low pages |

## Processes

| Term | Meaning |
|---|---|
| **Task / app** | One entry of the kernel app table; up to `MAXAPPS=16` (see [process model](../02-architecture/process-model.md)) |
| **Focus** | Exclusive right to read input and draw on the visible screen; held by one task (`focusappaddr`) |
| **Visual task** | Task that called `OS_SETGFX` and therefore participates in focus cycling (`fgfx` flag) |
| **Frozen task** | Task deactivated with `OS_FREEZEAPP`; keeps memory, is not scheduled |
| `factive`, `fchildfinished`, `fgfx`, `fwaiting` | App flag bits defined in [`sysdefs.asm`](../../NedoOS/src/_sdk/sysdefs.asm) |
| `YIELD` | Voluntary end of the current timeslice (`CMD_YIELD`); `YIELDKEEP` variant may be rescheduled within the same frame |
| `WAITPID` | Pattern: `OS_SETWAITING` + `YIELD` + `CMD_GETCHILDRESULT` to join a child |
| `safestack` | 18-byte per-task interrupt save area holding the full Z80 context |
| **idle task** | Always-existing task `app1`; mounts drives, draws the logo, launches `term.com`, polls the C+M+D hotkey |

## File systems & files

| Term | Meaning |
|---|---|
| **TR-DOS** | Classic Spectrum disk OS (ROM at `0x3D00`); NedoOS keeps compatibility through drivers ([TR-DOS driver](../03-kernel/filesystem-stack.md)) |
| **Hobeta** | Single-file TR-DOS container format (`.$B`, `.$C` files); produced by `hobeta.asm` / `nedotrd` |
| **TRD** | TR-DOS disk image format (640 KB floppy image) |
| **FCB** | CP/M File Control Block, 33 bytes in NedoOS (`FCB_sz=33` in `sysdefs.asm`) |
| **DTA** | Disk Transfer Address — buffer pointer for CP/M-style record I/O (`CMD_SETDTA`) |
| **FILINFO** | FatFS-style directory entry structure, 86 bytes with 8.3 and long names |
| **FatFS** | ChaN’s embedded FAT implementation, ported in [`fatfs4os`](../../NedoOS/src/fatfs4os) |
| **Volume** | Mounted unit with a drive letter; `vol_trdos=4`, `vol_pipe=25` are internal codes |
| **Pipe** | 255-byte ring buffer connecting two tasks (see [I/O & pipes](../02-architecture/io-and-pipes.md)) |
| `MAXPATH_sz` | 256 — maximum path string length |
| 8.3 | Short filename format: 8 name chars + 3 extension chars |

## Kernel & API

| Term | Meaning |
|---|---|
| **BDOS** | Basic Disk Operating System — the CP/M-inherited syscall gateway at `0x0005` (see [syscall interface](../03-kernel/syscall-interface.md)) |
| **RST vector** | One-byte Z80 restart call used as a cheap syscall: `0x00` QUIT, `0x08` GETKEY, `0x10` PRCHAR, `0x18/20/28` SETPG, `0x38` INT |
| `CMD_*` | Numeric BDOS function codes, defined in `sysdefs.asm` (0x05–0x22 CP/M, 0x43–0x5e MSX-DOS, 0xc6–0xff NedoOS) |
| `OS_*` | sjasmplus macros wrapping the calls with register setup (`sys_h.asm`) |
| **`callbdos_mutex`** | Per-task reentrancy guard around BDOS entry |
| `COMMANDLINE` | `0x0080` — length-prefixed command line of the running `.com` program |
| `PROGSTART` | `0x0100` — load and execution address of user programs |
| **`sys_timer`** | 32-bit 50 Hz frame counter (`CMD_GETTIMER`) |
| **key codes** | `key_esc`, `key_left`, `ext1..ext26`, `ss*`, `cs*` — symbolic names from the “Usable key codes” section of `sysdefs.asm` |
| `key_redraw` | Synthetic key (31) sent to a task when it regains focus |
| **gfx mode** | CRT controller value (`0xBD77`): 0 EGA / 2 MC / 3 6912 / 6 text |

## Networking

| Term | Meaning |
|---|---|
| **ZXNETUSB** | Community NIC: WIZnet W5300 hard-TCP/IP + SL811 USB host on one board |
| **W5300** | WIZnet hardwired TCP/IP chip; kernel driver [`w5300.asm`](../../NedoOS/src/kernel/w5300.asm) |
| **ESPNET** | ESP8266 firmware “coprocessor” over UART; driver [`espnet.asm`](../../NedoOS/src/kernel/espnet.asm), user lib `_sdk/espnet.asm` |
| `INETDRV` | Build-time network selector: 0 none, 1 WIZnet+SL811, 2 ESP8266 |
| **SOCKET** | 8-bit handle returned by `OS_NETSOCKET` |
| `sockaddr_in` | 15-byte address structure: family, big-endian port, 4-byte IPv4, zero pad |
| errno | BSD-style error numbers (`ERR_EAGAIN=35`, `ERR_NOTSOCK=38`, …) |
| **aynet** | AY-chip based network protocol experiments ([`aynet/proto.txt`](../../NedoOS/src/aynet/proto.txt)) |

## Applications & formats

| Term | Meaning |
|---|---|
| `.com` | Executable image loaded at `0x0100` (not MS-DOS MZ, just flat bytes) |
| `.bat` | Batch script for `cmd`, with `%0..%9` argument macros |
| `.$C` / `.$B` | Hobeta files: `.$C` code (auto-start), `.$B` boot file |
| **PT2/PT3** | Pro Tracker music module formats (AY) |
| **TFC** | TurboFM-compiled tracker format |
| **MOD** | Amiga ProTracker module, played on General Sound |
| **SNA** | ZX Spectrum memory snapshot (48K/128K), runnable under `nmisvc` |
| **TAP** | Tape image; `playtap` plays it to the real tape-out port |
| **Z-machine / zxzvm** | Infocom interactive-fiction VM; [`zxzvm`](../../NedoOS/src/zxzvm) is its NedoOS host |
| **NedoLang / NedoAsm** | Self-hosted language & assembler suite ([`nedolang`](../../NedoOS/src/nedolang)) |
| **Evo SDK** | SDCC-based game SDK used by `games/_sdk` |
| **MegaLZ** | LZ packer used for the kernel image (`mhmt -mlz`) |

## Build

| Term | Meaning |
|---|---|
| `syssets.asm` | Generated per-configuration include (`atm=`, `sys_npages`, `NEMOIDE`, `SYSDRV`, `INETDRV`, `PS2KBD`, `atm2clock`) |
| **sjasmplus** | Z80 assembler used for all assembly targets ([host tools](../06-tools-and-build/host-tools.md)) |
| **aspp** | Dependency generator for `.asm` files (a `gcc -MM` analogue) |
| **mhmt** | MegaLZ/Hrust packer, compresses `syscode.c` |
| **nedotrd / nedores / nedopad** | SDK host utilities: TRD image builder, resource converter, font pad editor |
| **dmimg** | Disk image manipulation tool (adddir etc.) |
| `release/` | Output tree: `bin/` executables, `doc/` manuals, platform images |
| **hobeta target** | Kernel packed as `nedoos.$C` boot file for HDD/SD installs |
