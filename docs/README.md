# NedoOS — Reverse-Engineering Documentation

Welcome to the reverse-engineered documentation set for **NedoOS**, the multitasking
operating system for ZX Spectrum compatibles (ATM Turbo 2, ATM3, ZX Evolution,
Pentagon 2.666). This documentation tree was produced by systematic study of the
source tree in [`NedoOS/src`](../NedoOS/src) — every statement is grounded in the
code, build scripts, and original project notes (`nedoos.txt`, `nedoos_en.md`,
`api_base.txt`, `api_net.txt`, `nedoos.new` changelog).

> The sources are largely commented in Russian (CP866/CP1251 encodings).
> This documentation set renders the essential content in English.

## Reading paths

| If you want to… | Read |
|---|---|
| Understand what NedoOS **is** and why it exists | [01-introduction/overview.md](01-introduction/overview.md) → [goals-and-requirements.md](01-introduction/goals-and-requirements.md) |
| Grasp the **big picture** of the design | [02-architecture/system-architecture.md](02-architecture/system-architecture.md) → [memory-map.md](02-architecture/memory-map.md) → [process-model.md](02-architecture/process-model.md) |
| Hack on the **kernel** | [03-kernel/kernel-structure.md](03-kernel/kernel-structure.md) → [syscall-interface.md](03-kernel/syscall-interface.md) → [filesystem-stack.md](03-kernel/filesystem-stack.md) |
| Write a **user application** | [04-sdk/sdk-overview.md](04-sdk/sdk-overview.md) → [api-catalog.md](04-sdk/api-catalog.md) → [coding-guidelines.md](04-sdk/coding-guidelines.md) |
| **Build** the system or produce images | [06-tools-and-build/build-system.md](06-tools-and-build/build-system.md) → [release-images.md](06-tools-and-build/release-images.md) |
| Learn what **software ships** with the OS | [05-applications/applications-overview.md](05-applications/applications-overview.md) and the per-category pages |
| Find where something **lives in the tree** | [07-appendix/source-map.md](07-appendix/source-map.md) |

## Full index

### [01 — Introduction](01-introduction/overview.md)

1. [Overview & reverse-engineering method](01-introduction/overview.md)
2. [Goals, functional requirements & constraints](01-introduction/goals-and-requirements.md)
3. [Hardware platforms & peripherals](01-introduction/hardware-platforms.md)
4. [Glossary of NedoOS & ZX Spectrum terms](01-introduction/glossary.md)

### [02 — Architecture](02-architecture/system-architecture.md)

1. [System architecture (layered view)](02-architecture/system-architecture.md)
2. [Memory map & banked memory management](02-architecture/memory-map.md)
3. [Process model: tasks, focus, scheduler](02-architecture/process-model.md)
4. [I/O model: pipes, stdio, terminals](02-architecture/io-and-pipes.md)
5. [Interrupts, timing, music & sampled sound](02-architecture/interrupt-and-timing.md)

### [03 — Kernel internals](03-kernel/kernel-structure.md)

1. [Kernel structure & boot sequence](03-kernel/kernel-structure.md)
2. [Syscall interface: RST vectors, BDOS dispatch](03-kernel/syscall-interface.md)
3. [Filesystem stack: FAT (FatFS), TR-DOS, volumes](03-kernel/filesystem-stack.md)
4. [Network stack: WIZnet W5300, ZXNETUSB, ESP8266](03-kernel/network-stack.md)

### [04 — SDK & API](04-sdk/sdk-overview.md)

1. [SDK overview: headers, macros, runtime libraries](04-sdk/sdk-overview.md)
2. [BDOS API catalog (function tables)](04-sdk/api-catalog.md)
3. [Assembly coding guidelines (self-hosting rules)](04-sdk/coding-guidelines.md)

### [05 — Applications](05-applications/applications-overview.md)

1. [Applications overview & catalog](05-applications/applications-overview.md)
2. [Shell (`cmd`), batch files, terminals (`term`, `netterm`)](05-applications/shell-and-terminals.md)
3. [File management (`nv`, `dmm`, disk utilities)](05-applications/file-management.md)
4. [Editors & viewers (`texted`, `view`, `scratch`, `more`, `man`)](05-applications/editors-and-viewers.md)
5. [Development tools (`basic`, Turbo Pascal, BDS C, NedoLang, disassemblers)](05-applications/development-tools.md)
6. [Multimedia (`player`, `modplay`, `pt`, `gp`, `playtap`)](05-applications/multimedia.md)
7. [Network applications (`browser`, `telnet`, `ping`, servers)](05-applications/network-apps.md)
8. [Archivers & system utilities](05-applications/archivers-and-utils.md)
9. [Games & the games SDK](05-applications/games.md)

### [06 — Tools & build](06-tools-and-build/build-system.md)

1. [Build system: Makefiles, hardware configurations](06-tools-and-build/build-system.md)
2. [Host-side tools (assembler, packers, image tools)](06-tools-and-build/host-tools.md)
3. [Release images: TRD, Hobeta, installation, emulator setup](06-tools-and-build/release-images.md)

### [07 — Appendix](07-appendix/history-and-credits.md)

1. [Project history & credits](07-appendix/history-and-credits.md)
2. [Source map: where things live](07-appendix/source-map.md)

## Conventions

* **Contributing / editing rules:** see [AGENTS.md](AGENTS.md) — the strict
  reverse-engineering, style, consistency and pre-commit checklist for this
  documentation set.
* Hexadecimal values are written `0x1234` (sjasmplus style, as in the sources);
  the original Russian docs use `#1234` — both mean the same.
* Z80 register pairs are written `DE`, `HL`, `IX` etc. All integers are
  little-endian unless stated otherwise.
* Paths are relative to the repository root, e.g.
  [`NedoOS/src/kernel/main.asm`](../NedoOS/src/kernel/main.asm).
* Diagrams are [Mermaid](https://mermaid.js.org/) blocks; GitHub and most
  Markdown viewers render them natively.
* Mermaid node labels avoid Cyrillic and special characters on purpose —
  some renderers choke on them.

## Upstream

NedoOS is developed in upstream Subversion (`svn://nedoos.ru/nedoos/nedoos`) by
Alone Coder (Dmitry Bystrov) and contributors; this repository mirrors it to Git
via a nightly sync (see [`NedoOS/scripts/sync-svn.py`](../NedoOS/scripts/sync-svn.py)
and [history-and-credits.md](07-appendix/history-and-credits.md)).
