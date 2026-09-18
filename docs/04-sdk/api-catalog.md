# API Catalog: Every System Call

*Prev: [SDK overview](sdk-overview.md) · Next: [Coding guidelines](coding-guidelines.md)*

This is the complete reverse-engineered reference of the NedoOS BDOS
interface, cross-indexing three sources that must always agree:

* [`_sdk/sysdefs.asm`](../../NedoOS/src/_sdk/sysdefs.asm) — the `CMD_*`
  function numbers and structure offsets;
* [`_sdk/sys_h.asm`](../../NedoOS/src/_sdk/sys_h.asm) — the documented
  `OS_*` wrapper macros (register contracts below come from there);
* the `tbdoscmds` jump table at the tail of
  [`kernel/main.asm`](../../NedoOS/src/kernel/main.asm) — the kernel side.

## How a call is made

```mermaid
sequenceDiagram
    participant A as Application
    participant U as userkrnl (page 0)
    participant K as Kernel (sysbdos)
    A->>A: macro sets L = CMD_*
    A->>U: rst 0x05 (BDOS)
    U->>U: out (0xfd),0x57 ; magic fd_system
    Note over U,K: 0x0000-0x3FFF now shows kernel page
    U->>K: jp BDOS_ label
    K->>K: dispatch via tbdoscmds[L]
    K-->>A: results in regs, Cy/A/L as error
```

Error-reporting conventions differ by family (a compatibility artefact):

* **CP/M-era calls** return counts/flags in `A` with no uniform errno;
* **MSX-DOS handle calls** return `A=0` on success, nonzero FRESULT-style
  error otherwise (and often carry Cy);
* **NedoOS invented calls** mostly return `A=0` for OK / `A!=0` for failure,
  except the socket family which returns negative `HL`/`L` + errno in `A`
  (see [network stack](../03-kernel/network-stack.md)).

## Function number allocation

| Range | Origin | Status |
|---|---|---|
| `0x05`–`0x22` | CP/M 2.2 | supported subset; several deviations (noted) |
| `0x43`–`0x5e` | MSX-DOS 1/2 | supported subset; `CMD_RENAME` deviates |
| `0xc6`–`0xff` | NedoOS | native extensions |
| anything else | — | returns with `A=1` (bad function) |

The gap between `0x23` and `0x42` and everything above `0xff` is deliberately
unused so future MSX-DOS/CP/M compatibility work has room.

## CP/M inherited calls (0x05–0x22)

The header comment in `sysdefs.asm` says it plainly: *"from CP/M (try to
avoid use!)"* — they exist for Turbo Pascal 3, BDS C and other CP/M-lineage
software. Deviations from real CP/M are marked ⚠.

| `L` | Macro | Contract |
|---|---|---|
| 0x05 | `OS_PRCHAR` (also `rst 0x10`) | `E`=char → console |
| 0x0e | `OS_SETDRV` | `E`=drive 0..14 → `A!=0` not mounted, `L`=drive count |
| 0x0f | `OS_FOPEN` | `DE`→unopened FCB |
| 0x10 | `OS_FCLOSE` | `DE`→opened FCB |
| 0x11 | `OS_FSEARCHFIRST` | `DE`→FCB with `?` wildcards, result copied to DTA |
| 0x12 | `OS_FSEARCHNEXT` | ⚠ **not CP/M compatible**: the wildcard FCB must be supplied on *every* call |
| 0x13 | `OS_FDEL` | **DEPRECATED** — use `CMD_DELETE` |
| 0x14 | `OS_FREAD` | read 128 bytes to DTA; ⚠ `A` = bytes actually read (not CP/M's keyed return) |
| 0x15 | `OS_FWRITE` | write 128 bytes from DTA |
| 0x16 | `OS_FCREATE` | `DE`→unopened FCB |
| 0x1a | `OS_SETDTA` | `DE`=data transfer address |
| 0x21 | `OS_RNDRD` | random record read; position = 3-byte record no. in FCB+21 (TP uses bytes 21,22) |
| 0x22 | `OS_RNDWR` | random record write, same layout |

`rst 0x08` (`OS_GETKEY` — wait for key in `A`) and `rst 0x10` (print char in
`A`) are shortcut vectors alongside the BDOS family; see
[syscall interface](../03-kernel/syscall-interface.md).

## MSX-DOS inherited calls (0x43–0x5e)

Handle-based, path-oriented — the preferred file API for portable code.

| `L` | Macro | Contract |
|---|---|---|
| 0x43 | `OS_OPENHANDLE` | `DE`→ASCIIZ path, `A`=mode (b0 no-write, b1 no-read, b2 inheritable) → `B`=handle, `A`=error |
| 0x44 | `OS_CREATEHANDLE` | like open plus `B`=required attrs (b7 = fail-if-exists) |
| 0x45 | `OS_CLOSEHANDLE` | `B`=handle → `A`=error |
| 0x48 | `OS_READHANDLE` | `B`=handle, `DE`→buffer, `HL`=length → `HL`=bytes read, `A`=error(0=ok) |
| 0x49 | `OS_WRITEHANDLE` | mirror of read |
| 0x4a | `OS_SEEKHANDLE` | `B`=handle, `DE:HL`=signed offset, `A`=method 0/1/2 = begin/cur/end |
| 0x4d | `OS_DELETE` | `DE`→ASCIIZ path → `A`=error |
| 0x4e | `OS_RENAME` | `DE`→old path, `HL`→new path. ⚠ **not MSX-DOS compatible**: new name is a *full path* — this is how move-across-directories works |
| 0x5a | `OS_CHDIR` | `DE`→ASCIIZ dir → `A`=error |
| 0x5c | `OS_PARSEFNAME` | `DE`→dot-name → `HL`→11-byte CP/M name; *NOT RECOMMENDED*, kept for TP |
| 0x5e | `OS_GETPATH` | `DE`→256-byte buffer → filled with `d:/path/` (drive-prefixed), `HL`→last path item |

`OS_SEEKHANDLE` return is the new position in `DE:HL`; combine with
`CMD_TELLHANDLE` and `CMD_GETFILESIZE` below for full random I/O.

## NedoOS native calls (0xc6–0xff)

Grouped by purpose; entries marked ⚠ OBSOLETE are kept only so old binaries
run — new code must not use them.

### Process & tasking

| `L` | Macro | Contract |
|---|---|---|
| 0xc6 | `OS_PUTKEY` | `E`=code, `D`=0 letter / 1 control / 2 = SS+Enter gfx switch → `A`=0 ok, 1 queue full |
| 0xcd | `OS_GETCHILDRESULT` | `HL`←result of last waited-for child |
| 0xd1 | `OS_HIDEFROMPARENT` | detach: parent no longer waits for me; `HL`=result code at exit |
| 0xdf | `OS_DROPAPP` | `E`=id — kill task (what `drop` uses) |
| 0xe0 | `OS_GETAPPMAINPAGES` | `E`=id → `D,E,H,L`=pages at 0x0000/4000/8000/C000, `C`=flags |
| 0xec | `OS_CHECKPID` | `E`=id → `A!=0` if it is *my child* and alive |
| 0xed | `OS_FREEZEAPP` | `E`=id — stop scheduling it, drop its gfx |
| 0xf3 | `OS_RUNAPP` | `E`=id — (re)activate a disabled app |
| 0xf4 | `OS_NEWAPP` | make new *disabled* app → `B`=id, `DEHL`=its pages; loader then fills code & calls RUNAPP |
| 0xf2 | `OS_YIELD` | cooperative switch — **use the macro, never `halt`** |
| 0xff | `OS_YIELDKEEP` | like YIELD but keeps me first in queue (used inside kernel loops) |
| 0xd9 | `OS_SETWAITING` | mark current task waiting (internal; used by wait loops) |

### Memory & pages

| `L` | Macro | Contract |
|---|---|---|
| 0xca | `OS_GETMEMPORTS` | → `IX,BC,DE,HL`=port values for windows 0/4000/8000/C000 |
| 0xcb | `OS_GETPAGEOWNER` | `E`=page → `E`=owner id (0 free, 0xFF system) |
| 0xe9 | `OS_SETMAINPAGE` | `E`=page for my `0x0000` window |
| 0xfb | `OS_GETMAINPAGES` | → `D,E,H,L`=my pages, `C`=flags, `B`=my id |
| 0xfc | `OS_NEWPAGE` | → `A`=0 ok, `E`=fresh page |
| 0xfd | `OS_DELPAGE` | `E`=page — give a page back to the OS pool |

### Files & directories (native extensions)

| `L` | Macro | Contract |
|---|---|---|
| 0xcf | `OS_OPENDIR` | `DE`→path — open directory stream into per-task slot |
| 0xd0 | `OS_READDIR` | `DE`→`FILINFO` buffer; `FILINFO.fname[0]=0` means end |
| 0xda | `OS_GETFILESIZE` | `B`=handle → `DE:HL`=size |
| 0xe5 | `OS_TELLHANDLE` | `B`=handle → `DE:HL`=current offset |
| 0xe8 | `OS_GETFILINFO` | `DE`→path, `HL`→FILINFO buffer (stats without opening) |
| 0xe3 | `OS_GETFILETIME` | `DE`→path → `IX`=date, `HL`=time |
| 0xe4 | `OS_SETFILETIME` | `DE`→path, `IX`=date, `HL`=time (what `touch` does) |
| 0xeb | `OS_MKDIR` | `DE`→ASCIIZ path → `A`=error |
| 0xef | `OS_MOUNT` | `E`=drive — (re)scan & mount that volume |
| 0xea | `OS_SETSYSDRV` | switch *system* drive → `A!=0` not mounted, `L`=count |

`FRESULT` codes returned in `A` by this family (from the `sysdefs.asm`
comment on `CMD_RENAME`): 0 `FR_OK`, 1 `DISK_ERR`, 2 `INT_ERR`,
3 `NOT_READY`, 4 `NO_FILE`, 5 `NO_PATH`, 6 `INVALID_NAME`, 7 `DENIED`,
8 `FR_EXIST`, 9 `INVALID_OBJECT`, 10 `WRITE_PROTECTED`, 11 `INVALID_DRIVE`,
12 `NOT_ENABLED`, 13 `NO_FILESYSTEM`, 14 `MKFS_ABORTED`, 15 `TIMEOUT`,
16 `LOCKED`, 17 `NOT_ENOUGH_CORE`, 18 `TOO_MANY_OPEN_FILES` — i.e. the
FatFS result space, passed through by the volume layer.

### Console & graphics

| `L` | Macro | Contract |
|---|---|---|
| 0xd2 | `OS_SETSTDINOUT` | `B`=task id, `E`/`D`/`H`=stdin/stdout/stderr handles (how pipes are wired) |
| 0xd3 | `OS_GETSTDINOUT` | → `E`/`D`/`H`=handles, `L`=stdout height in lines |
| 0xd8 | `OS_SETBORDER` | `E`=0..15 |
| 0xf6 | `OS_CLS` | `E`=colour — clear screen |
| 0xf7 | `OS_SETCOLOR` | `E`=colour byte for subsequent text |
| 0xf8 | `OS_SETXY` | `DE`=yx — move cursor |
| 0xe1 | `OS_GETXY` | ⚠ OBSOLETE → `DE`=current yx |
| 0xf5 | `OS_PRATTR` | ⚠ OBSOLETE — attribute at cursor |
| 0xee | `OS_GETATTR` | **DEPRECATED** — read attribute at cursor |
| 0xe6 | `OS_SCROLLUP` | ⚠ OBSOLETE (TEXTMODE only), `DE`=top yx, `HL`=hgt,wid |
| 0xe7 | `OS_SCROLLDOWN` | ⚠ OBSOLETE mirror |
| 0xc7 | `OS_GETGFX` | → `A`=raw gfxmode (`#BD77`), `B`=focus id, `C`=screen 0/1, `D..L`=pgscr0/1 |
| 0xf9 | `OS_SETGFX` | `E`=mode: 0 EGA, 2 MC, 3 6912, 6 text; +8 noturbo, +0x80 keep pages; `E=-1` disable (returns old mode) |
| 0xfa | `OS_SETPAL` | `DE`→32-byte palette |
| 0xc8 | `OS_GETPAL` | `DE`←32-byte palette |
| 0xfe | `OS_SETSCREEN` | `E`=0/1 — select active screen buffer |

### Time, sound & misc

| `L` | Macro | Contract |
|---|---|---|
| 0xc9 | `OS_SETTIME` | `IX`=date, `HL`=time (word formats as GETTIME) |
| 0xe2 | `OS_GETTIME` | → `IX`=date, `HL`=time; drives the clock & file timestamps |
| 0xf1 | `OS_GETTIMER` | → `DE:HL`=frames since boot (50 Hz tick counter) |
| 0xd4 | `OS_PLAYCOVOX` | `HL`→sample data at 0xC000+ (0x00 ends), `DE`→page table at 0x0000+, `HX`=delay — 18≈11 kHz, 7≈22 kHz, 1≈44 kHz |
| 0xd5 | `OS_SETMUSIC` | `HL`→module at 0x4000..0x7FFF (0 = stop), `A`=page for 0x8000 window |
| 0xcc | `OS_GETCONFIG` | → `H`=system drive, `L`=platform (1 Evo, 2 ATM2, 3 ATM3, 6 p2.666), `E`=pgsys, `D`=TR-DOS page, `IXBC`=SVN revision |
| 0xd6 | `OS_READSECTORS` | `B`=drive, `DE`→buffer, `IX:HL`=32-bit sector, `A`=count → `A`=error |
| 0xd7 | `OS_WRITESECTORS` | mirror of read |
| 0xf0 | `OS_GETKEYMATRIX` | → `BCDEHLIX` = halfrows CS…Space (raw matrix) |

### Network (CMD_WIZNETOPEN family)

Full treatment with subfunction table in the
[network stack](../03-kernel/network-stack.md#the-socket-api) page:

| `L` | Macro | Contract |
|---|---|---|
| 0xdb | `OS_NETSOCKET` … | `L`=subfunction 1..0x0b (socket, connect, bind, listen, accept, shutdown, DNS, UART cfg, info) |
| 0xdd | `OS_WIZNETREAD` | TCP: `A`=sock, `DE`→buf, `HL`=len; UDP/ICMP add `DE`→sockaddr → `HL`=count (negative = errno in `A`) |
| 0xde | `OS_WIZNETWRITE` | mirror of read |
| 0xdc | `OS_WIZNETCLOSE` | `A`=sock, `E`=0 full / 1 tx-only |

## Data structures

### FILINFO (directory entry, 86 bytes)

| Offset | Size | Field |
|---|---|---|
| 0 | 4 | `FSIZE` — 32-bit file size |
| 4 | 2 | `FDATE` — last modified date |
| 6 | 2 | `FTIME` — last modified time |
| 8 | 1 | `FATTRIB` — attribute (`FATTRIB_DIR 0x10` = directory) |
| 9 | 13 | `FNAME` — 8.3 short name with dot, ASCIIZ |
| 22 | 64 | `LNAME` — long name buffer (`DIRMAXFILENAME64`), ASCIIZ |

Returned by `OS_READDIR`/`OS_GETFILINFO`; when `LNAME[0]=0` display
`FNAME` instead.

### FCB (33 bytes, CP/M-compatible superset)

| Offset | Field | Notes |
|---|---|---|
| 0 | `drv` | drive number |
| 1 | `FNAME` | 11 bytes, CP/M `NAMEEXT` form |
| 12 | `EXTENTNUMBERLO` | not used |
| 13 | `FATTRIB` | attribute byte |
| 14 | `EXTENTNUMBERHI` | not used |
| 15 | `RECORDCOUNT` | not used |
| 16 | `FSIZE` | dword |
| 20 | `FTIME` | word |
| 22 | `FFSFCB` | word — TR-DOS FCB or FIL pointer |
| 24 | `DIRPOS` | word — directory scan position |
| 28 | `RECORDSIZE` | word — must be 128 |
| 30 | `FDATE` | word |
| 32 | `FRECORD` | byte — current record for sequential access |

The NedoOS-specific fields (`FFSFCB`, `DIRPOS`, `FSIZE`, timestamps) ride in
CP/M's *reserved* area — binaries built for real CP/M still lay out correctly.

### Date & time words

`IX`=date, `HL`=time use the FAT packing (same as MS-DOS): date =
`(year-1980)<<9 | month<<5 | day`; time = `hour<<11 | min<<5 | sec/2`.

## Key codes (what GETKEY returns)

Codes 1..26 = Ext-letter pseudo-keys (`extA..extZ`); `cs0..cs9` = 8,
`0xf4..0xfc` = Ctrl/CS-digit combos; `ss*` symbols for SS-modified keys.
Frequently used aliases:

| Constant | Value | Meaning |
|---|---|---|
| `key_enter` | 13 | Enter |
| `key_backspace` | 8 | = `cs0` |
| `key_tab` | 9 | = `csss` |
| `key_esc` | 27 | = `csspace` |
| `key_left/right/up/down` | `cs5/cs8/cs7/cs6` | arrows |
| `key_pgup/pgdown` | `cs3/cs4` | page keys |
| `key_home/end/ins` | `ssQ/ssE/ssW` | Home / End / Ins |
| `key_del` | `cs9` | Delete |
| `key_F1..key_F10` | `ext1..ext9, ext0` | function keys |
| `key_csenter` | 0xfd | CS+Enter — gfx mode switch |
| `key_redraw` | 31 | synthetic "repaint" event |
| `NOKEY` | 0 | no key |

The full table (`ext*`, `ss*`, `cs*` bases) is in `sysdefs.asm`
*"Usable key codes"*; raw halfrow state comes from `OS_GETKEYMATRIX`.

## Reserved / unassigned

* `0xce` — `CMD_RESERV_1`, reserved for future use.
* Handle bits: `PIPEADD80` marks a pipe handle, `TRDOSADD40` a TR-DOS handle
  (see [I/O & pipes](../02-architecture/io-and-pipes.md)).

## See also

* [SDK overview](sdk-overview.md) — macros vs call-level libraries.
* [Syscall interface](../03-kernel/syscall-interface.md) — dispatch internals.
* [api_base.txt](../../NedoOS/src/_sdk/api_base.txt) /
  [api_net.txt](../../NedoOS/src/_sdk/api_net.txt) — the canonical essays.
