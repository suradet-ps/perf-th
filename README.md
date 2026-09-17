# perf-th

```
██████╗ ███████╗██████╗ ███████╗              ████████╗██╗  ██╗
██╔══██╗██╔════╝██╔══██╗██╔════╝              ╚══██╔══╝██║  ██║
██████╔╝█████╗  ██████╔╝█████╗      █████╗       ██║   ███████║
██╔═══╝ ██╔══╝  ██╔══██╗██╔══╝      ╚════╝       ██║   ██╔══██║
██║     ███████╗██║  ██║██║                      ██║   ██║  ██║
╚═╝     ╚══════╝╚═╝  ╚═╝╚═╝                      ╚═╝   ╚═╝  ╚═╝
```

---

## ◆ PULSE

[![GitHub Pages](https://img.shields.io/badge/Pages-live-2ea44f)](https://suradet-ps.github.io/perf-th/)
[![License](https://img.shields.io/badge/license-MIT%20OR%20Apache--2.0-blue.svg)](#-anatomy)

A profile has a hot path, and an optimization has a measurement - perf-th
is the Thai bridge to both. This is the complete Thai translation of the
official Rust Performance Book: 20 chapters built with mdbook, terminology
locked by a single glossary, and every code block byte-identical to the
original. The links are checked against the built book (199 anchors), the
structure mirrors the upstream repo file-for-file, and the license travels
with the text. Built for the Thai-speaking Rustacean:
[suradet-ps.github.io/perf-th](https://suradet-ps.github.io/perf-th/).

| 20 chapters translated ▣ | Glossary 77 terms ▣ | Links 340/340 ▣ | Build passing ▣ |
|---|---|---|---|

*v1.0.0 - translation, glossary, verification, and the static build
are all sealed.*

> Built with mdbook 0.5 + Markdown, translated from
> [nnethercote/perf-book](https://github.com/nnethercote/perf-book),
> verified by script and rendered as static HTML - a book with the
> flamegraph on the page.
>
> **suradet-ps**, artifact keeper

---

## ◆ IGNITION

One runtime, three commands.

```
⟫ git clone https://github.com/suradet-ps/perf-th.git
⟫ cd perf-th
⟫ cargo install mdbook
⟫ mdbook serve --open
```

Open [http://localhost:3000](http://localhost:3000).

```
⟫ mdbook build                                # static HTML into book/
⟫ powershell scripts/check-links.ps1          # all anchors in the built book (pwsh on Linux/macOS)
⟫ powershell scripts/verify-translation.ps1   # byte-exact check vs upstream
```

> On Linux or macOS, run the verification scripts using `pwsh scripts/<script>.ps1`.
> `verify-translation.ps1` checks against `nnethercote/perf-book` in adjacent directories or via `-Orig <path>`.

<details>
<summary>Translating a chapter</summary>

A chapter is a file: `src/<chapter>.md`, listed in `src/SUMMARY.md`.
The glossary lives in `GLOSSARY.md` - a term is chosen once and reused
everywhere. Code blocks, commands, links, and filenames stay verbatim;
only prose and headings are translated. Heading anchors follow mdbook's
slug rules (Thai tone marks are stripped), so anchors are copied from the
built HTML, never guessed.

</details>

---

## ◆ ANATOMY

One stack, zero custom JS, several quiet helpers.

- **Translates** - the complete book: the introduction, benchmarking,
  build configuration, linting, profiling, inlining, hashing, heap
  allocations, type sizes, standard library types, iterators, bounds
  checks, I/O, logging and debugging, wrapper types, machine code,
  parallelism, general tips, and compile times - Thai prose over
  untouched code.
- **Glossaries** - `GLOSSARY.md` locks the vocabulary (77 terms, one
  Thai term per concept, chosen once and reused), so chapter seventeen
  agrees with chapter two.
- **Verifies** - `scripts/verify-translation.ps1` diffs every code block
  (62 of them), heading level, and link target (340 of them) against
  upstream `nnethercote/perf-book` - byte-exact or it does not pass.
- **Checks** - `scripts/check-links.ps1` walks the built book and
  resolves every anchor link against real heading ids - 199 of them,
  all reachable.
- **Builds** - mdbook renders static HTML into `book/`, zero server
  runtime, readable offline and searchable by built-in static index.
- **Licenses** - MIT OR Apache-2.0, inherited from upstream, with the
  LICENSE files shipped beside the text.

---

## ◆ RITUALS

**The core ceremony** - the translation pass:

1. Open a chapter in `src/`. The upstream `nnethercote/perf-book` repo
   sits beside it (clone `https://github.com/nnethercote/perf-book`
   alongside `perf-th`) - structure is a contract.
2. Translate the prose; keep every code block and command as the
   original wrote it.
3. Consult `GLOSSARY.md` for every term that already has a canon.
   New terms get proposed in the glossary first.
4. Build, verify, check. The book builds clean, the diff is
   byte-exact, and the anchors resolve.

**The ceremony of the anchor** - mdbook slugs strip Thai tone marks and
keep vowel signs, so a heading's anchor is never its plain spelling.
Anchors are read from the built HTML, written into the source, and
re-verified - a guessed anchor is a broken link waiting to happen.

**The ceremony of the code block** - a translated command that is not
byte-identical to the original is a regression, not a translation.
The verifier is the conscience of the repo.

---

## ◆ ECHOES

**Where this artifact is heading**

```
P1 ▸ SUMMARY, title page, introduction ─────────────────────────────── ▸ sealed
P2 ▸ benchmarking, build configuration, compile times ──────────────── ▸ sealed
P3 ▸ linting and profiling ─────────────────────────────────────────── ▸ sealed
P4 ▸ hashing, heap allocations, type sizes, library types, wrappers ── ▸ sealed
P5 ▸ inlining, iterators, bounds checks, I/O, logging, machine code ── ▸ sealed
P6 ▸ parallelism and general tips ──────────────────────────────────── ▸ sealed
P7 ▸ glossary, license, link verification, mdbook build ────────────── ▸ sealed
```

**Raising the artifact** - the honest path lives in `GLOSSARY.md`
(term canon), `scripts/` (the verification gate), and `book.toml`
(book config). New chapters follow the frontmatter-free contract of the
SUMMARY. Open an issue first to discuss a change.

**Status** - on every change: `mdbook build` must pass, the translation
verifier must report byte-exact code blocks across all 21 files
(20 chapters + `SUMMARY.md`), and the link checker must report
`ALL ANCHOR LINKS OK`.
[Watch the gates](scripts).

---

```
  ─────────────────────────────────────────
   Every profile has its hot path
   Every book has its first page
  ─────────────────────────────────────────
```

Translated from the [Rust Performance Book](https://github.com/nnethercote/perf-book),
which is licensed under [MIT](LICENSE-MIT) OR [Apache-2.0](LICENSE-APACHE).
