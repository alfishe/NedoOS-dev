# Archivers and Utilities

*Prev: [Network applications](network-apps.md) · Next: [Games](games.md)*

Sources: [`pkunzip/`](../../NedoOS/src/pkunzip), [`tar/`](../../NedoOS/src/tar),
[`unrar/`](../../NedoOS/src/unrar), [`zxrar/`](../../NedoOS/src/zxrar),
[`crc/`](../../NedoOS/src/crc), [`winto866/`](../../NedoOS/src/winto866),
[`calc/`](../../NedoOS/src/calc), [`ztst/`](../../NedoOS/src/ztst),
[`menu/`](../../NedoOS/src/menu), [`scrshot/`](../../NedoOS/src/scrshot),
[`print/`](../../NedoOS/src/print), kapps utilities, and the manual.

## Archive support matrix

```mermaid
flowchart LR
    subgraph in["Unpacking (in-tree)"]
        PZ[pkunzip] -->|".zip .gz"| CUR[current directory]
        TA[tar] -->|".tar"| CUR
        UR[unrar] -->|".rar 2.x"| CUR
    end
    subgraph out["Packing (in-tree)"]
        TA2[tar] -->|file or dir → "*.tar"| TARB["archive"]
        ZR[zxrar] →|"→ mynewrar.rar"| RARB[".rar 2.x"]
        KZ[kapps/pkzip] →|".zip"| ZPB["archive"]
    end
```

* **pkunzip** — unpacks whole `*.zip` / `*.gz` archives into the current
  directory. The single most-used importer of files from the PC world.
* **tar** — dual-purpose: unpacks `*.tar` to the current directory; given a
  non-archive argument, *packs* that file (or a whole directory tree) into
  an archive named after it with the extension replaced by `.tar`.
* **unrar** — interactive `*.rar` (2.x) extractor: `v` view contents,
  `e` extract selected files, `m` enter a filename prefix mask.
* **zxrar** — the matching *packer*: reads a filename from the command
  line and creates/adds to `mynewrar.rar`.
* **kapps/pkzip** — IAR C implementation for creating `*.zip`.

## Checksums: crc / md5

[`crc/`](../../NedoOS/src/crc) computes checksums and hashes:
`crc.asm` (CRC32 of files), [`md5.asm`](../../NedoOS/src/crc/md5.asm)
(MD5), with test vectors in `tst_md5/` and alternative implementations
(`speh/`, `md5_speh_tool/`) cross-checked during development. The `NOTES`
file records the verification protocol — checksums matter when your
transport is a 30-year-old floppy.

## Text & console utilities

| Tool | Purpose |
|---|---|
| [`winto866`](../../NedoOS/src/winto866) | convert Windows-1251/UTF-8 text files to the console's CP866 |
| [`setfont`](../../NedoOS/src/setfont) | install console fonts (CP866/CP1125/ATM) — see [editors](editors-and-viewers.md) |
| [`print`](../../NedoOS/src/print) | hard-copy utility: prints file/screen contents to a printer; politely `OS_HIDEFROMPARENT`s and releases unused pages first |
| [`scrshot`](../../NedoOS/src/scrshot) | saves the current EGA screen (both screen pages, 32K at `0x4000`) to a file — screenshots for documentation |
| [`freetime`](../../NedoOS/src/freetime) | small stdio/command-loop utility (auxiliary demo) |
| [`shay`](../../NedoOS/src/shay) | AY silencer (see [multimedia](multimedia.md)) |
| [`reset`](../../NedoOS/src/reset) | soft reset |

## calc — the calculators

[`calc/`](../../NedoOS/src/calc) is a console calculator family (80-column
text UI, command line at row 24) shipped in three editions:
`common/` (basic four-function), `conversion/` (unit conversion tables)
and `extended/` (scientific). Over 100 source files make it one of the
largest pure-asm applications in the tree; `cmdpr.asm` + `str.asm` are
shared with the shell's line editor.

## menu — retro menu shell

[`menu/`](../../NedoOS/src/menu) is a ported/reverse-engineered
Soviet-PC-style **menu shell**: it reads `.mnu` menu descriptions, draws
boxed windows (`twind.asm`, `wtro.asm`) with the Iskra-style `Font42_f.cod`
font (`ttyp42.asm`), and launches files via a `extent.txt`-style
association table — the same *spirit* as `nv.ext` but from the Iskra
lineage. Included as both nostalgia and a second shell demonstrating the
OS's neutrality to shells.

## ztst — Z80 validation suite

[`ztst/`](../../NedoOS/src/ztst) ("A collection of different Z80 tests
ported to NedoOS"): **ZEXDOC** (documented flags), **ZEXALL**
(incl. undocumented behaviour), `z80test` and `test_z80_emu` — the
reference exercisers for anyone bringing up an emulator or an FPGA core;
under NedoOS they run as ordinary tasks and print through stdio.

## kapps utilities (IAR C)

| App | Purpose |
|---|---|
| [`calendar`](../../NedoOS/src/kapps/calendar) | calendar view |
| [`tm`](../../NedoOS/src/kapps/tm), [`time2`](../../NedoOS/src/kapps/time2) | time tools (C cousins of `time`) |
| [`sleep`](../../NedoOS/src/kapps/sleep) | delay utility for scripts |
| [`dhrystone`](../../NedoOS/src/kapps/dhrystone) | the classic Dhrystone benchmark — Z80 vs the world |
| [`multibank`](../../NedoOS/src/kapps/multibank) | multi-page memory test/demo |
| [`grep`](../../NedoOS/src/kapps/grep) | text search across files |
| [`updater`](../../NedoOS/src/kapps/updater) | network self-update helper |
| [`hobeta`](../../NedoOS/src/kapps/hobeta), [`rdtrd2`](../../NedoOS/src/kapps/rdtrd2), [`wrtrd2`](../../NedoOS/src/kapps/wrtrd2), [`deltree`](../../NedoOS/src/kapps/deltree) | file tools (see [file management](file-management.md)) |
| [`aes`](../../NedoOS/src/kapps/aes) | AES cipher implementation/demo |
| [`emptyres`](../../NedoOS/src/kapps/emptyres) | resource-file template for kapps |

## Utility patterns worth copying

* `print` and `scrshot` both **release their second screen page**
  (`OS_GETMAINPAGES` + `OS_DELPAGE`) before doing anything — the canonical
  memory-frugal console app.
* `scrshot` snapshots `user_scr0_low/high` mirrors before switching
  graphics mode — read those variables, not the ports (see
  [memory map](../02-architecture/memory-map.md)).
* Everything here is a plain `.com` using only public BDOS calls — no
  kernel privileges anywhere in userland.

## See also

* [File management](file-management.md) — where unpacked files go.
* [Build system](../06-tools-and-build/build-system.md) — the host-side
  packers that produce release images.
* [Games](games.md) — the fun side of the same loader.
