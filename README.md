# Course Syllabus Template

A modular LaTeX system for producing **accessible, tagged (PDF/UA) course
syllabi**. Content that is shared across every course you teach (university
branding, instructor information, and boilerplate policy statements) lives in a
central `globals/` directory, while each course keeps only its own content in a
self-contained `courses/<course>/` directory. Updating a university policy or
your office hours once propagates to every syllabus that uses it.

A complete, compilable example lives in [`courses/sample/`](courses/sample/)
(ECON 2105, Principles of Macroeconomics) — see
[`courses/sample/syllabus.pdf`](courses/sample/syllabus.pdf) for the rendered
output.

## Features

- **Accessible output.** Built with `\DocumentMetadata` and LaTeX tagging
  (`pdfstandard=ua-2`) to produce a tagged, PDF/UA-oriented document with proper
  reading order and MathML for math content.
- **Separation of shared vs. course-specific content.** Edit a policy statement
  or your contact details once in `globals/`; every course picks up the change.
- **Multi-institution branding.** Each institution directory under `globals/`
  carries its own logo, brand colors, and instructor block; a course chooses its
  institution by which files it inputs.
- **Per-course content as small includes.** Each syllabus section (description,
  topics, learning outcomes, grading, schedule, etc.) is a separate file under
  the course's `includes/` directory.

## Repository structure

```
globals/
  preamble.tex          % shared LaTeX preamble: fonts, packages, colors,
                        %   list environments, heading styles, tagging
  frontmatter.tex       % title block (\maketitle), PDF metadata, running headers
  uga/                  % an institution profile (add more as needed)
    institution.tex     %   \institution, logo (\brandmark), brand colors
    instructor.tex      %   instructor macros: \profname, \profemail, ...
    statements/         %   reusable policy text shared across that institution's courses
      academichonesty.tex
      accommodation.tex
      resource_wellbeing.tex
      resource_success.tex
      resources_ai.tex
      disclaimer.tex

courses/
  sample/               % a single course
    syllabus.tex        %   driver file: sets course macros and assembles the document
    includes/           %   this course's section content
      coursedesc_catalog.tex
      coursedesc_topics.tex
      coursedesc_learningoutcomes.tex
      coursedesc_gened.tex
      attendance.tex
      materials.tex
      grading.tex
      schedule.tex
```

> **Note on tracked files.** `.gitignore` is scoped so that only the shared
> globals (`preamble.tex`, `frontmatter.tex`, and `globals/uga/`) and the
> `courses/sample/` example are tracked. Your own institutions and live course
> directories stay private by default, and LaTeX build artifacts
> (`*.aux`, `*.log`, `*.fls`, `*.fdb_latexmk`, `*.synctex.gz`, `*-mathml.html`)
> are ignored at any depth.

## Requirements

- A recent TeX distribution — **TeX Live 2024 or newer** (or current MiKTeX) —
  for `\DocumentMetadata` and PDF tagging support.
- The **LuaLaTeX** engine (required by `fontspec`/`unicode-math` and the tagging
  setup).
- Fonts: **Source Serif 4**, **Source Sans 3**, **Source Code Pro**, and **New
  Computer Modern Math**. The Source families are installed system-wide from
  [Adobe's Source repositories](https://github.com/adobe-fonts); New Computer
  Modern ships with TeX Live.

## Building a syllabus

Compile from inside the course directory (the driver uses paths relative to it):

```sh
cd courses/sample
lualatex syllabus.tex      # run twice to resolve the running header and references
```

Or, with `latexmk`:

```sh
cd courses/sample
latexmk -lualatex syllabus.tex
```

The result is `syllabus.pdf` in that directory.

## How a course is assembled

Each `courses/<course>/syllabus.tex` is the driver. In order, it:

1. Sets `\DocumentMetadata` and the document class.
2. Inputs an institution profile and instructor block, e.g.
   `\input{../../globals/uga/institution}` and `.../instructor`.
3. Inputs the shared `\input{../../globals/preamble}`.
4. Defines the course's own macros — `\semester`, `\coursenum`, `\coursename`,
   `\credithours`, `\callnumber`, `\prereq`, `\coreq`, `\location`,
   `\classtimes`, textbook fields, etc.
5. Inputs `\input{../../globals/frontmatter}` to build the title and headers.
6. In the body, lays out the sections, pulling course content with
   `\input{includes/...}` and shared policies with
   `\input{../../globals/uga/statements/...}`.

## Adding a new course

1. Copy `courses/sample/` to `courses/<your-course>/`.
2. In `syllabus.tex`, update the course macros (term, number, name, meeting
   times, textbook, etc.) and confirm the institution/instructor/statement
   `\input` paths point to the institution you want.
3. Edit the files in `includes/` with your course's content.
4. Compile as above.

## Adding or switching an institution

1. Create `globals/<institution>/` with `institution.tex` (set `\institution`,
   `\brandmark` logo path, `\brandmarkwidth`, and the `\brandhex*` colors),
   `instructor.tex`, and a `statements/` directory.
2. In a course's `syllabus.tex`, change the institution `\input` lines (and the
   statement `\input` paths in the body) to the new directory.

## Credits & license

This project began as a fork of James Quinlan's LaTeX syllabus template and has
since been substantially restructured into the modular, multi-course,
accessibility-focused system documented above. Released under the
[MIT License](LICENSE).
