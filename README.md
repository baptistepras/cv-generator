# LaTeX CV Generator (`cv-generator`)

A single-page, client-side website to **build a CV in your browser** and export it as:

- **LaTeX source** (view / copy / download `.tex`)
- **PDF preview** (compiled online, then downloadable)
- An **Overleaf project draft** (opens Overleaf with the generated LaTeX)

Everything is done from `index.html` (no framework): the UI edits a structured state, then generates LaTeX from it.

---

## Table of contents

- [Quick start](#quick-start)
- [What you can do on the website](#what-you-can-do-on-the-website)
  - [1) Customize the theme color](#1-customize-the-theme-color)
  - [2) Toggle dark / light mode](#2-toggle-dark--light-mode)
  - [3) Fill personal information](#3-fill-personal-information)
  - [4) Add CV sections](#4-add-cv-sections)
  - [5) Add entries inside sections](#5-add-entries-inside-sections)
  - [6) Reorder sections and entries](#6-reorder-sections-and-entries)
  - [7) Insert page breaks](#7-insert-page-breaks)
  - [8) Generate LaTeX](#8-generate-latex)
  - [9) Compile & preview as PDF](#9-compile--preview-as-pdf)
  - [10) Open in Overleaf](#10-open-in-overleaf)
  - [11) Load an example CV](#11-load-an-example-cv)
  - [12) Report a bug / request a feature](#12-report-a-bug--request-a-feature)
- [How it works (technical overview)](#how-it-works-technical-overview)
  - [Data model](#data-model)
  - [Rich text → LaTeX conversion](#rich-text--latex-conversion)
  - [LaTeX template & section rendering](#latex-template--section-rendering)
  - [PDF compilation](#pdf-compilation)
  - [Overleaf export](#overleaf-export)
- [Notes / limitations](#notes--limitations)

---

## Quick start

1. Open the website https://baptistepras.github.io/cv-generator/.
2. Fill **Personal Info** (top block).
3. Add sections (Education, Experience, Projects, etc.).
4. Click **Compile & Preview** to generate a PDF.
5. Optionally click **View LaTeX** to copy/download the `.tex`, or **Open in Overleaf**.

---

## What you can do on the website

### 1) Customize the theme color

In the top bar, the **Color** palette lets you choose the accent color used in the generated CV.

Available presets:

- Midnight (default)
- Navy
- Forest
- Crimson
- Plum

This color becomes the LaTeX `themecolor` (and is applied to headings / name when not default black).

### 2) Toggle dark / light mode

Click the sun/moon icon in the top bar to switch the **website UI** between:

- Dark mode
- Light mode

This only affects the editor interface (not the compiled PDF theme, which is controlled by the accent color).

### 3) Fill personal information

The **Personal Info** block contains:

- Full name
- Title
- Email
- Website
- GitHub URL
- LinkedIn URL

Each field is a small rich-text editor: you can format text with:

- **Bold**, *Italic*, Underline
- Text color (small swatches)

These values are used to generate the LaTeX header:

- Name + subtitle (title)
- Contact lines with icons (email / website / GitHub / LinkedIn)
- Website is linked as `https://<website>` (so you typically enter `domain.com`)

### 4) Add CV sections

At the bottom, use the **Add section bar**:

- `+ Education`
- `+ Experience`
- `+ Publications`
- `+ Projects`
- `+ Skills`
- `+ Achievements`
- `+ Languages`
- `+ Generic`
- `↵ Page break`

Each section appears as a card and is also listed in the **sidebar** for quick navigation.

### 5) Add entries inside sections

Most sections contain **entries**. Each entry is a collapsible card:

- The header shows a *preview* (taken from the first field of that entry, e.g., Degree/Role/Title/etc.)
- Click the entry header to collapse/expand the entry editor
- Remove an entry via the **Remove** button in the entry footer

Supported section types and their fields:

#### Education
- Degree / Program
- Institution
- Note (e.g. honors)
- Date range
- Location

#### Experience
- Role / Title
- Organization
- Date range
- Location
- Description (long rich text; supports bullet lists)
- Contact (optional)

If “Contact” is filled, it is rendered as an italic line `Contact: ...` in the LaTeX output.

#### Publications
- Title
- Venue

#### Projects
- Title
- URL (plain input field)
- Description (long rich text; supports bullet lists)

The URL is used to render a clickable link icon in the generated CV.

#### Skills
- Category
- Items (long rich text; supports bullet lists)

#### Achievements
- Achievement (single rich text line per entry)

#### Languages
- Language
- Level
- Certifications (optional)

Languages are rendered in a **two-column layout** (two entries per row).
Certifications are split into multiple lines if you separate them with:
- a dash (`-`)
- an en dash (`–`)
- an em dash (`—`)
- or newlines

#### Generic
- Content (long rich text; supports bullet lists)

This is useful for fully custom blocks (e.g., “Other projects”, “Interests”, etc.).

### 6) Reorder sections and entries

You can reorder in two ways:

**Sections**
- Drag sections using the drag handle (⋯/grip icon) to reorder visually
- Or use the sidebar **Up / Down** arrow buttons

**Entries**
- Drag entries inside a section to reorder them

The website syncs the DOM order back into the internal state before exporting.

### 7) Insert page breaks

Add a **Page break** section to force a `\newpage` in the generated LaTeX.

You can reorder page breaks like any other section (drag them or move them via sidebar arrows).

### 8) Generate LaTeX

Click **View LaTeX** (bottom action bar) to open a modal containing the generated `.tex`.

From there you can:
- **Copy** the LaTeX to clipboard
- **Download .tex**

### 9) Compile & Preview as PDF

Click **Compile & Preview** to open the PDF modal and compile the LaTeX using online compilation services.

What you can do there:
- See compilation status
- View the resulting PDF in an embedded viewer
- Download the compiled `cv.pdf`

If compilation fails, a **Debug log** toggle appears to help diagnose errors / network failures.

### 10) Open in Overleaf

Click **Open in Overleaf** to open a new Overleaf tab and send the generated LaTeX using a form POST.

This is helpful if:
- Online compilation APIs are unreachable
- You want to further tweak the CV in a full LaTeX editor
- You want to manage images, packages, bibliography, etc.

### 11) Load an example CV

Click **Load example** in the top bar to auto-fill:
- Personal info fields
- A complete multi-section CV (education, publications, experience, projects, skills, achievements, languages)
- A sample page break

This is the fastest way to understand the expected formatting and section structure.

### 12) Report a bug / request a feature

Click **Report** (🐞) to open a form where you can submit:

- Your name (optional)
- Your email (optional)
- Type: Bug report / Feature request / Other
- Description (required)

The form is sent via Formspree. A small disclaimer explains email usage.

---

## How it works (technical overview)

### Data model

The website maintains a simple in-memory state:

- `state.sections[]` where each section has:
  - `id`
  - `type` (education, experience, project, etc.)
  - `name` (editable section title)
  - `entries[]` each with `id` and `fields{...}`

The UI renders from that state and updates it on edit.

### Rich text → LaTeX conversion

Most text fields are `contenteditable` rich text areas.
When exporting, the HTML is converted into LaTeX:

- `<b>/<strong>` → `\textbf{}`
- `<i>/<em>` → `\textit{}`
- `<u>` → `\underline{}`
- `<span style="color:...">` → `\textcolor[HTML]{...}{...}`
- `<ul><li>...</li></ul>` → `itemize` list
- Special characters are escaped (`& % $ # _ { } ~ ^ \` etc.)

### LaTeX template & section rendering

The generator builds a full LaTeX document:
- A4 page, tight margins
- Header with name/title + contact lines
- `\section{...}` blocks for each section name
- Dedicated macros for education, experience, projects, etc.
- Optional theme color (`themecolor`) if accent color isn’t the default

A “Page break” section inserts `\newpage`.

### PDF compilation

When you request a PDF preview, the website:
1. Generates LaTeX
2. Tries to compile via `latex.ytotech.com`
3. If that fails due to network-type errors, it falls back to `texlive.net`
4. If both are unreachable, it tells you to use **Open in Overleaf**

### Overleaf export

The website posts the generated LaTeX to Overleaf (`encoded_snip`) and opens it in a new tab.

---

## Notes / limitations

- This is a **single-file** web app (no persistence): reloading the page resets your CV unless you copy/download the LaTeX.
- Rich text editing is intentionally minimal (bold/italic/underline, colors, and bullet lists on “long” fields).
- PDF compilation depends on third-party services availability; use Overleaf as the reliable fallback.
- Website field “Website” is treated as a domain and linked with `https://` automatically.

---
