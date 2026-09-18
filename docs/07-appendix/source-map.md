# Source Map: Repository Inventory

*Up: [Documentation hub](../README.md) · Next: [History & credits](history-and-credits.md)*

Every directory of the workspace, one line each, with pointers into the
design documents. Paths are relative to the workspace root
(`NedoOS-dev/`).

## Workspace root

| Path | What it is |
|---|---|
| [`NedoOS/`](../../NedoOS) | the OS sources (git submodule → github.com/alfishe/NedoOS, synced from SVN `svn://nedoos.ru`) |
| [`docs/`](.) | this reverse-engineering documentation set |
| `tools/`, `.gitmodules`, `.gitignore` | workspace scaffolding |

## `NedoOS/` root

| Path | Contents |
|---|---|
| `.svn-revision` | upstream SVN revision of this snapshot (**r2685**) |
| [`src/`](../../NedoOS/src) | all OS code (below) |
| [`tools/`](../../NedoOS/tools) + [`tools/src/`](../../NedoOS/tools/src) | host toolchain binaries & sources ([host tools](../06-tools-and-build/host-tools.md)) |
| [`us/`](../../NedoOS/us) | UnrealSpeccy reference emulator station + ROMs |
| [`release/`](../../NedoOS/release) | build outputs: `os*.trd`, `*.$C`, `bin/ doc/ ini/ nedogame/ nedodemo/` ([release images](../06-tools-and-build/release-images.md)) |
| [`scripts/sync-svn.py`](../../NedoOS/scripts/sync-svn.py), `.github/workflows/sync-svn.yml` | nightly SVN→Git sync |

## `src/` top-level files

| File | Role |
|---|---|
| [`Makefile`](../../NedoOS/src/Makefile) | master build: targets `atm2 atm2hd atm3 atm3hd atm3sd evolution pe26`, `trd`, `hobeta`, `syssets-*` ([build system](../06-tools-and-build/build-system.md)) |
| [`README.linux`](../../NedoOS/src/README.linux) | the authoritative Linux build instructions (Russian) |
| `Makefile.doc.en`, `Makefile.hlp` | documentation / on-system help builds |
| [`nedoos_en.md`](../../NedoOS/src/nedoos_en.md) (+`.ctex`) | the official English manual — ground truth for app behaviour |
| `nedoos.txt`, `nedoos.new`, `nedoos.thk` | Russian manual, changelog, notes ([history](history-and-credits.md)) |
| [`net.ini`](../../NedoOS/src/net.ini), `autoexec.bat`, `BOOT6000.$B` | shipped config / boot files |
| `make.bat`, `makeall.bat`, `mkatm2*.bat` … `mkpe26sd.bat`, `builddir.bat`, `clean*.bat` | Windows build entry points |

## `src/` directories

### Kernel & core

| Dir | Contents | Docs |
|---|---|---|
| [`kernel/`](../../NedoOS/src/kernel) | 38 files: `main.asm` boot+dispatch, `syskrnl.asm` scheduler, `sysbdos.asm` BDOS core, `userkrnl.asm` trampoline, `idle.asm`, `sysfile.asm`/`sysfs.asm`, `trdosfs.asm`+`trdosio.asm`, `fatfsdrv.asm`, `w5300*.asm`, `espnet*.asm`, `sl811.asm`, `syskey*.asm`, `sysfonts.asm`, `syspal.asm`, `say.asm`, `ptsplay.asm`, … | [kernel structure](../03-kernel/kernel-structure.md) + friends |
| [`fatfs4os/`](../../NedoOS/src/fatfs4os) | ChaN FatFS port: `ff.c ff.h ffconf.h integer.h`, glue `mylib.asm savelij.asm`, `fatfs.raw` | [filesystem](../03-kernel/filesystem-stack.md) |
| [`_sdk/`](../../NedoOS/src/_sdk) | headers (`sys_h.asm`, `sysdefs.asm`), runtime libs, `api_*.txt`, tool sources (`nedotrd/ nedores/ nedopad/ convega/`), `common.mk`, `iar.mk`+`iar.lib`, codepage tables, logos | [SDK](../04-sdk/sdk-overview.md) |

### System-side applications

| Dir | Contents | Docs |
|---|---|---|
| `cmd` | shell | [shell](../05-applications/shell-and-terminals.md) |
| `term`, `netterm`, `more`, `man`, `reset` | console stack | [shell](../05-applications/shell-and-terminals.md) |
| `nv` (+`nv.ext`) | Nedovigator file manager | [files](../05-applications/file-management.md) |
| `texted` | text editor | [editors](../05-applications/editors-and-viewers.md) |
| `view` | NedoView image viewer | [editors](../05-applications/editors-and-viewers.md) |
| `scratch` | EGA graphics editor | [editors](../05-applications/editors-and-viewers.md) |
| `setfont` | console fonts | [editors](../05-applications/editors-and-viewers.md) |
| `basic` | NedoBasic | [dev tools](../05-applications/development-tools.md) |
| `tp`, `bdsc` | Turbo Pascal 3, BDS C | [dev tools](../05-applications/development-tools.md) |
| `nedolang` | NedoLang/NedoAsm self-hosting suite (`comp/ asm/ tok/ …`, ARM `1986VE1T`) | [dev tools](../05-applications/development-tools.md) |
| `z80`, `x86`, `bk`, `vic20`, `zxzvm`, `nmisvc`, `untr` | debuggers & emulators | [dev tools](../05-applications/development-tools.md) |
| `player`, `pt`, `modplay`, `gp`, `playtap`, `rcpplay`, `shay` | music software | [multimedia](../05-applications/multimedia.md) |
| `browser`, `wget`*, `telnet`, `ping`, `myip`, `scrnet` | network clients/servers | [network apps](../05-applications/network-apps.md) |
| `pkunzip`, `tar`, `unrar`, `zxrar` | archivers | [utils](../05-applications/archivers-and-utils.md) |
| `crc`, `winto866`, `calc`, `ztst`, `menu`, `scrshot`, `print`, `freetime` | utilities | [utils](../05-applications/archivers-and-utils.md) |
| `mktrd`, `rdtrd`, `wrtrd`, `gettrd` | TR-DOS image tools | [files](../05-applications/file-management.md) |
| `hello`, `emptyapp` | program templates | [SDK](../04-sdk/sdk-overview.md) |
| `demos` | `gfxtest`, `mandelbr`, `noise`, `raytrace`, `showay`, `testV9990` — graphics/sound demos incl. V9990 chip experiments | — |
| `aynet` | AY-port networking experiment (`proto.txt`, `yad/`) | [network](../03-kernel/network-stack.md#historical--experimental) |
| `moon-rabbit-zx`, `mrabbit-fusion` | Moon Rabbit platformer (base + Fusion editions) | [games](../05-applications/games.md) |
| `games` | 47 games + `_sdk` Evo SDK | [games](../05-applications/games.md) |
| `kapps` | ~40 IAR C apps + `common/`, `iarlib/` (see below) | [overview](../05-applications/applications-overview.md) |
| `gp` extra | contains `ngsdec/`, `ptsplay/`, `vgm/`, `mbwave/`, `moonmod/`, `moonmid/`, `common/` subsystems | [multimedia](../05-applications/multimedia.md) |

\* `wget` sources live in the upstream tree (dmapps-adjacent); binary ships in `release/bin`.

### `kapps/` (IAR C) quick index

`aes ansiview atelnet calendar cdplay common cuart deltree dhrystone dns
emptyres enet espcfg getpic girc gopher grep gstest hobeta iarlib
multibank nc nco netprint ngsplay pkzip playwav rdtrd2 scaview sleep
svnesp sxgview textview tgvplay time2 tm updater wrtrd2 zxart-radio
zxdb zifi` — categorized in
[network apps](../05-applications/network-apps.md),
[utils](../05-applications/archivers-and-utils.md),
[multimedia](../05-applications/multimedia.md),
[files](../05-applications/file-management.md).

### Referenced but absent from this snapshot

`dmapps/*` (`settime 3ws dmftp dmirc dmm helloworld`), `nmisvc` and
`ngsdec` as a
top-level dir appear in `src/Makefile` `SUBDIRS`; their sources did not
make it into this SVN snapshot — binaries ship in `release/bin`
([build system](../06-tools-and-build/build-system.md)).

## Suggested reading order in the sources

1. [`_sdk/api_base.txt`](../../NedoOS/src/_sdk/api_base.txt) →
   [`_sdk/sys_h.asm`](../../NedoOS/src/_sdk/sys_h.asm) — the contracts.
2. [`kernel/main.asm`](../../NedoOS/src/kernel/main.asm) — boot & BDOS
   table.
3. [`kernel/syskrnl.asm`](../../NedoOS/src/kernel/syskrnl.asm) —
   scheduler & app table.
4. [`term/term.asm`](../../NedoOS/src/term/term.asm) +
   [`cmd/cmd.asm`](../../NedoOS/src/cmd/cmd.asm) — how a real console app
   behaves.
5. [`nv/nv.asm`](../../NedoOS/src/nv/nv.asm) — a full GUI app.

## See also

* [Documentation hub](../README.md).
* [History & credits](history-and-credits.md).
