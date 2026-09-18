# AGENTS.md — Rules for Documenting NedoOS

Strict working rules for anyone (human or AI agent) editing this
documentation set. Follow them exactly; deviations degrade the set's core
value — that **every statement is verifiable against the sources**.

## Prime directive

The docs describe the **actual code**, not intentions, memories, or
upstream folklore. If you cannot point to a file/line, a build artifact,
or an official manual section (`nedoos_en.md`, `nedoos.txt`, `api_*.txt`,
`nedoos.new`) supporting a claim, the claim does not go in.

## 1. Reverse-engineering method

1. **Read before writing.** Open the real file (Read/Grep/LSP) for every
   fact you add or edit — never rely on memory of a previous session.
2. **Decode sources properly.** NedoOS sources and notes are **CP866 or
   Windows-1251**, never UTF-8. Convert with
   `iconv -f cp866 -t utf-8` when quoting. Never write UTF-8 back into
   `NedoOS/src/**`.
3. **Triangulate API facts.** A syscall contract is true only if
   `src/_sdk/sysdefs.asm` (number), `src/_sdk/sys_h.asm` (macro +
   register contract) and the dispatch table in
   `src/kernel/main.asm`/`sysbdos.asm` (implementation) agree.
4. **Separate fact classes** and label them:
   * *documented* — stated in the manual or `api_*.txt`;
   * *observed in code* — cite the file;
   * *inferred* — mark explicitly ("evidenced by…", "apparently").
5. **Absent ≠ nonexistent.** Directories referenced by `src/Makefile`
   but missing from the checkout (e.g. `dmapps/*`, `nmisvc`) must be
   marked "not present in this snapshot" — never silently documented
   from imagination.
6. **Numbers are evidence.** Constants, ports, addresses, key codes,
   sizes come from the sources verbatim (`grep` them); no rounding, no
   "approximately" where an exact value exists.
7. Re-verify anything you touch: if an edit changes a stated count
   (apps, games, links, files), recount.

## 2. Documenting style

* **English only**; identifiers, file names and register names verbatim.
* One topic per file; prefer a new small page over growing a monolith.
  Target ≤ ~300 lines per page; split when it exceeds ~400.
* Page skeleton (keep uniform):
  1. `# Title` — `*Prev: … · Up: … · Next: …*` navigation line;
  2. `Sources:` block linking the evidence files;
  3. body sections;
  4. `## See also` with 2–4 cross-links.
* Link to sources as `../../NedoOS/src/...` relative paths from section
  dirs (`docs/NN-x/...`), `../NedoOS/...` from `docs/README.md`.
* Prose wrapped at ~78 columns. Tables over long prose for catalogs.
* Register-contract notation: `E`=value, `DE`→pointer, `HL`=result,
  `A=0`=ok / `A!=0`=error. Hex as `0x1234`; keep original `#1234` only
  inside verbatim quotes.
* Terminology must match [01-introduction/glossary.md](01-introduction/glossary.md);
  add new terms there first.

## 3. Mermaid diagrams

* Use where structure genuinely helps (flows, layering, timelines,
  sequence). Not as decoration; do not duplicate a table right below.
* **Label safety** — no Cyrillic, no unescaped quotes or `#` in labels;
  `<br/>` for line breaks. Common typographic marks (`—`, `→`, `×`)
  render fine; anything beyond that, stay ASCII.
* Only well-supported diagram types: `flowchart`, `sequenceDiagram`,
  `stateDiagram-v2`, `timeline`, `pie`, `mindmap` (mindmap sparingly).
* Wrap every fenced block correctly; an unclosed fence breaks the page.

## 4. Editing mechanics (chunking)

To prevent tool timeouts and allow review:

* New file: initial Write ≤ ~250 lines.
* Extensions: SearchReplace adding ~150–250 lines; combined
  original+new text per call < ~400 lines.
* Prefer more small checkpoints over few large ones.
* SearchReplace anchors must be unique and match whitespace exactly.
* Never rewrite a whole existing file to make a small change.

## 5. Consistency contract

* **README.md is the hub**: every new page must be added to its Full
  Index (and Reading paths if it opens a topic). `AGENTS.md` itself is
  exempt from indexing.
* Sibling pages in a section form a Prev/Next chain; update both
  neighbours' headers when inserting a page.
* Cross-links between pages are relative (`../02-architecture/...`).
  Anchors (`#section`) only for stable headings.
* Don't invent categories — place new content into the existing
  seven-section layout (`01-introduction` … `07-appendix`).
* Keep counts cited in prose (e.g. "31 markdown files") in sync, or
  phrase them so they don't rot ("~40 IAR C apps").

## 6. Pre-commit checklist

Run **all** of these before committing; fix everything they report.

1. **Link integrity** — every relative link resolves:
   ```sh
   cd docs && python3 - <<'EOF'
   import os, re, glob
   root = os.getcwd(); broken = []
   for md in glob.glob(root + "/**/*.md", recursive=True):
       for m in re.finditer(r'\]\(([^)#\s]+)(?:#[^)\s]*)?\)',
                            open(md, encoding="utf-8").read()):
           t = m.group(1)
           if t.startswith(("http://", "https://", "mailto:")): continue
           if not os.path.exists(os.path.normpath(
                   os.path.join(os.path.dirname(md), t))):
               broken.append((os.path.relpath(md, root), t))
   print("broken:", broken or "none")
   EOF
   ```
2. **Index completeness** — every `docs/**/*.md` except `AGENTS.md` is
   mentioned in `docs/README.md`.
3. **Fences balanced** — even count of ` ``` ` lines per file.
4. **Mermaid sanity** — blocks parse; labels free of Cyrillic,
   unescaped quotes/`#` (typographic marks `— → ×` are allowed).
5. **Encoding hygiene** — no mojibake in quoted Russian text; no UTF-8
   files written under `NedoOS/src/`.
6. **Facts re-verified** — every claim you added/edited traces to a
   source file (spot-check links open the right file).
7. **Scope** — `git status` shows only intended paths under `docs/`;
   the `NedoOS` submodule stays untouched.
8. **Commit message** — `docs: ...` prefix, summary of what and where.

## 7. When facts and docs disagree

Code wins. Update the doc, and if the disagreement is significant (API
contract, boot behaviour), add a short *"Deviation note"* so readers who
knew the old story see what changed. Never "fix" the code from the docs.
