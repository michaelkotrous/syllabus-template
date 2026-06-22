# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A modular LaTeX system for producing **accessible, tagged (PDF/UA) course syllabi**. Shared content (university branding, instructor info, boilerplate policy statements) lives in `globals/`; each course keeps only its own content in a self-contained `courses/<course>/` directory. There is no application code, tests, or lint — the "build" is a LaTeX compile.

## Building

Compile **from inside the course directory** — the driver uses paths relative to it (`../../globals/...`):

```sh
cd courses/sample
lualatex syllabus.tex      # run TWICE: the running header and refs need a second pass
# or:
latexmk -lualatex syllabus.tex
```

- The engine **must be LuaLaTeX** (`fontspec`/`unicode-math` + the tagging setup require it; pdflatex/xelatex will not work).
- Requires **TeX Live 2024+** (for `\DocumentMetadata` and tagging) and the fonts **Source Serif 4 / Source Sans 3 / Source Code Pro** plus **New Computer Modern Math**.
- `courses/sample/` is the canonical working example to compile when verifying changes.

## Architecture

A course's `syllabus.tex` is a **driver** that assembles the document; order matters:

1. `\DocumentMetadata{...}` (sets PDF/UA tagging) then `\documentclass`.
2. `\input` an institution profile + instructor block (e.g. `../../globals/uga/institution`, `.../instructor`) — **before** the preamble, because the preamble reads brand-color macros defined there.
3. `\input{../../globals/preamble}` — shared packages, fonts, colors, list/heading styles, tagging.
4. Define the course's own `\def` macros (`\semester`, `\coursenum`, `\coursename`, `\credithours`, `\callnumber`, `\prereq`, `\coreq`, `\location`, `\classtimes`, textbook fields, `\equivalent`, …) — these must come **before** frontmatter.
5. `\input{../../globals/frontmatter}` — builds `\maketitle`, PDF metadata, and `fancyhdr` running headers from those macros.
6. Body: section headings interleaved with `\input{includes/...}` (course content) and `\input{../../globals/<inst>/statements/...}` (shared policy text).

Key consequence: **instructor info and policy statements are institution-global.** Editing `globals/uga/instructor.tex` or a file in `globals/uga/statements/` changes every course that inputs them, not just one. Course-specific text belongs in that course's `includes/`.

Switching a course to a different institution = changing the institution/instructor `\input` lines and the statement `\input` paths in the body; a `globals/<institution>/` profile supplies `institution.tex` (`\institution`, `\brandmark` logo, `\brandmarkwidth`, `\brandhex*` colors), `instructor.tex`, and `statements/`.

## Conventions / gotchas

- **Lists:** use the custom `ul` and `ol` environments (defined in `globals/preamble.tex`), **not** `itemize`/`enumerate`. The standard environments don't tag correctly; the custom ones are configured for the tagged-PDF output.
- **Accessibility is a hard requirement**, not a nice-to-have. Don't introduce constructs that break tagging or the PDF/UA standard.
- **Macros that auto-fill across courses:** prefer referencing existing course macros (e.g. `\coursenum`) inline rather than hardcoding values, so content updates when the course changes. (Example: the email subject-line note in the sample's instructor table references `\coursenum`.)
- **Empty-field handling:** optional table rows are guarded with `\ifdefempty{\macro}{}{...}`; leave a `\def\macro{}` empty rather than deleting it.
- **Math** uses `unicode-math` with `NewComputerModernMath`.

## Git / .gitignore

- Only the **shared globals** (`globals/preamble.tex`, `globals/frontmatter.tex`, `globals/uga/`) and the **`courses/sample/`** example are tracked. Other course dirs (e.g. `econ101_*`, `econ4020_*`, `econ4460_*`) and `globals/coastal/` exist on disk but are intentionally untracked/private.
- The **Core LaTeX ignore block must stay at the end of `.gitignore`**, after the `!`-prefixed un-ignore rules. Git applies the last matching pattern, so placing `*.aux` etc. earlier lets the `!courses/sample/*` rules re-include build artifacts. Bare globs (`*.aux`) already match at any depth.
- Build artifacts already committed won't be ignored until removed from the index (`git rm --cached`); `.gitignore` only affects untracked files.
