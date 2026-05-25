# Feather Fables — Project Handoff Document

> Drop this file in the project root on the new machine and tell the new Claude instance to read it. It contains everything needed to pick up where the current session left off.

## 1. Project overview

**Goal:** Produce a coffee-table photography book titled *Feather Fables* by P. Radhakrishna Pai (Mathru Kripa, Vittla). The book has two halves:

- **Life Essays section (pages 1–26):** Foreword-style essays from family, friends, and admirers about the author and his birding journey. This part is currently fully built in LaTeX.
- **Bird Stories section (page 27 onward):** Short photo+text pieces about specific bird sightings. ~52 articles exist in the original source; **only the first 10 are rendered in LaTeX so far**.

**Format:** A4 landscape (29.7 × 21 cm), two-column body, 12pt EB Garamond serif, ~1.45 line spacing.

**Final output:** `Feather Fables - Life Essays.pdf` — currently 47 pages.

## 2. Files in the project root (`/home/l1/gg/`)

### Source / authoritative
- `Sample Layout English Final.cdr` — Original CorelDRAW file (300 MB).
- `Sample Layout English Final English.pdf` — PDF export from CDR (485 MB). **This is the source of truth for all bird article text.** Use `pdftotext` to extract.
- `Feather Fables.docx`, `Feather Fables v2.docx`, `Feather Fables v3.docx` — older DOCX iterations (kept for history only).

### Active LaTeX build
- `Feather Fables - Life Essays.tex` — **The main LaTeX source.** Edit this directly.
- `Feather Fables - Life Essays.pdf` — Latest compiled output (47 pages).
- `Feather Fables - Life Essays.tex.bak`, `.tex.bak2` — Pre-change backups from major edits.
- `Feather Fables - Life Essays.aux`, `.log`, `.out`, `.toc` — XeLaTeX build artefacts.

### Assets
- `Untitled.png` — Cover image (full-bleed photograph on page 1 of the PDF).
- `Vrathik2gb/Vrathik/` — 251-file image folder. Mix of bird photos, wildlife, and travel/landmark shots. **Filenames like `5 s.jpg`, `frame 8.jpg`, `frame s 22.jpg` do not correspond to story numbers.** Identification was done by viewing each image.
- Essay portrait files in project root:
  - `daaj.jpeg` — Author + Daaji (landscape, two faces).
  - `twinsister.jpeg` — Shraddha + Shreya Pai Phadnis (near-square).
  - `anuradha.jpeg` — Anuradha Pai (590×1280, very tall portrait; needs `width=` sizing and yshift to centre the face).
  - `subra.jpeg` — P. Subraya Pai (portrait headshot).
  - `ullas_kamath.jpeg` — K. Ullas Kamath (portrait).
  - `judge.jpeg` — Jayaprakash V. (935×1593, tall portrait).
  - `skm.jpeg` — Shridhar Kodakkal (portrait headshot).
  - `z.jpeg` — M. Ananthakrishna Hebbar (6048×4032 landscape, but EXIF-rotated content — needs `angle=90` in `\includegraphics`).

### Documentation / list files
- `birds_list.txt` — Plain-text list of all 52 bird articles (title — species — location).
- `PROJECT_MEMORY.md` — This file.

## 3. Build environment

- **Compiler:** XeLaTeX (required for `fontspec`).
- **TeX distribution:** TeXLive (Arch Linux: `texlive-fontsextra` is needed for EB Garamond).
- **Fonts (system-installed):**
  - **EB Garamond** (body) — loaded via explicit `fontspec` path; not in standard fontconfig. The `.tex` uses `Extension=.otf, UprightFont=*-Regular` pattern.
  - **Inter** (headings, TOC, page numbers)
  - **Noto Serif Devanagari** (Sanskrit verse on page 6)

### Build command
```bash
cd /home/l1/gg
xelatex -interaction=nonstopmode "Feather Fables - Life Essays.tex"
```

Run twice for cross-references and the TOC `\pageref{bird:N}` labels to resolve correctly. Currently the build emits a few `Overfull \vbox` warnings around the bird pages — harmless.

### Rendering pages for visual verification
```bash
pdftoppm -r 80 -f <N> -l <N> "Feather Fables - Life Essays.pdf" /tmp/page -jpeg
```
Then `Read` the resulting `/tmp/page-NN.jpg` (Claude is multimodal — image read works).

## 4. LaTeX preamble: key macros

Defined near the top of `Feather Fables - Life Essays.tex`:

### `\attribution{text}`
Simple italic, right-aligned attribution under an essay (no portrait).

### `\portraitsquare{size}{img-opts}{yshift}{path}`
Square-cropped portrait inside a TikZ rectangle node with a thin charcoal border (`draw=ink!30`, `line width=0.4pt`). Uses `path picture` to clip an oversized image to the square. `yshift` is the vertical offset applied inside the clip (negative pushes the image down → exposes more of the *top* of the photo, useful for portrait shots where the face is in the upper half).

### `\attribphoto{size}{img-opts}{yshift}{path}{attribution-text}`
Wraps `\portraitsquare` in a right-aligned minipage with attribution below. Used *after* `\end{multicols}`. **Will be pushed to next page if there isn't room** — that turned out to be the most common pagination headache.

### `\attribphotoraw{img-opts}{path}{attribution-text}`
Same idea but **uses the photo at its natural aspect ratio** (no square clip). Used for daaj.jpeg (landscape, two faces) and subra.jpeg (head shot) where the square crop loses too much.

### Workaround for pagination problems
For essays where `\attribphoto` overflows to the next page even though there's empty space on the current page, the fix that worked was to **inline the portrait + attribution inside the `multicols` environment**, before `\end{multicols}`. The pattern:

```latex
... last paragraph of essay ...

\par\vspace{0.8em}
\hfill\portraitsquare{4.5cm}{height=4.7cm}{0.2cm}{twinsister.jpeg}

\par\itshape\raggedleft Written by: ...
\end{multicols}
```

This is used for pages 15 (twin sisters), 17 (Subraya), 23 (Hebbar), 25 (Kodakkal). The portrait then flows naturally with the right column and the orphan-page problem disappears.

For `daaj.jpeg` (page 9) and a few others, `\attribphoto` *after* `\end{multicols}` works fine because the essay is short.

## 5. Layout decisions baked in

### TOC structure
The TOC uses `longtable` (not `minipage` — that silently dropped entries 09-13 when content overflowed). Groups: OPENING, REFLECTIONS, FAMILY VOICES, TRIBUTES & FOREWORDS, WINGS WORDS & WONDER. Bird entries use `\pageref{bird:N}` so page numbers auto-update.

### Chapter titles
30 pt italic display serif, centred, with a thin charcoal rule and a `❦` (U+2766 floral heart) ornament below:
```latex
\titleformat{\section}{\centering}{}{0pt}
  {\fontsize{30pt}{36pt}\selectfont\itshape}
  [...rules... ❦ ...rules...]
```

### Sanskrit verse (page 6)
Uses `polyglossia` with `\setotherlanguage{sanskrit}` and `Noto Serif Devanagari` font. The page has a manual `\vspace*{-0.4cm}` to pull it up — that's intentional per user feedback.

### Bird pages (27 onward)
- 12 cm × 11.5 cm photo placeholder (TikZ dashed rectangle, label "PHOTO") on one side, 11.5 cm text column on the other.
- Photo position **alternates** between left and right by bird index (`bird:N` for N=1..10).
- Long bodies are auto-split: paragraph-aware splitter at ~700 char threshold, with the tail rendered full-width on the next page.
- Bird stories also have `\label{bird:N}` and TOC entries with `\pageref{bird:N}` so the TOC auto-updates.

### Cover page
Full-bleed `Untitled.png` via TikZ `remember picture, overlay` — the image fills the entire A4-landscape page.

## 6. Bird image matching (status: 7 of first 10)

Done in an earlier session by viewing every unique file in `Vrathik2gb/Vrathik/` and identifying species.

| # | Bird | Image file | Status |
|---|------|------------|--------|
| 1 | Yellow-browed Bulbul | `6 s.jpg` | ✅ inserted |
| 2 | Indian Grey Hornbill | `frame_l_67.jpg` (renamed from `frame l  67.jpg` — double-space breaks LaTeX) | ✅ inserted |
| 3 | Crested Bunting | `s16.jpg` | ✅ inserted |
| 4 | Common Green Magpie | — | ❌ not in folder; placeholder remains |
| 5 | Flame-throated Bulbul | `14 s.jpg` | ✅ inserted |
| 6 | (multi-species diary) | — | placeholder (not needed) |
| 7 | Red-bellied Leiothrix | `frame 93.jpg` | ⚠️ inserted but uncertain — actually likely Red-tailed Minla; close family, same Sattal region |
| 8 | Oriental Pied Hornbill | `s21.jpg` | ✅ inserted |
| 9 | Common Emerald Dove | — | ❌ not in folder; placeholder remains |
| 10 | Grey-headed Bulbul | `frame 8.jpg` | ✅ inserted |

The image-insertion logic is anchored to `\label{bird:N}` in the LaTeX. The script that did it: `/tmp/insert_bird_images.py` (since cleared from `/tmp`; recreate from scratch if needed — see `Feather Fables - Life Essays.tex` lines around the bird sections for the current state).

### Gotcha: filenames with double spaces
LaTeX collapses double spaces in `\includegraphics{...}` paths. `frame l  67.jpg` was copied to `frame_l_67.jpg` to work around this.

## 7. Essay portraits (status: all 7 done)

| Page | Essay | Portrait file | Shape | Notes |
|------|-------|---------------|-------|-------|
| 9 | Blessed Moments with Daaji | daaj.jpeg | **natural** (landscape rect, 4 cm tall) | shows both faces |
| 15 | From My Twin Daughters | twinsister.jpeg | **square 6.5 cm** | inside multicols; user wanted maximum size |
| 16 | My Life Partner | anuradha.jpeg | square 3.5 cm | needs `\enlargethispage{2cm}`; user accepted small-face crop because photo is full-body |
| 17 | My Elder Brother's Endeavour | subra.jpeg | **natural** (portrait rect, 4 cm tall) | inside multicols; "With Love" line added above name |
| 21 | My Observation | ullas_kamath.jpeg | square 4 cm | |
| 23 | Dva Suparna... | z.jpeg | **natural** (rotated 90° via `angle=90`, 3 cm tall) | inside multicols |
| 25 | Pai's This Venture... | skm.jpeg | **natural** (portrait rect, 4 cm tall) | inside multicols |
| 26 | Nature Through the Judge's Eyes | judge.jpeg | square 4 cm | `yshift=-1.4cm` to bring face down from a full-body shot |

### yshift cheat sheet
- For TikZ `path picture` clip: **negative yshift moves the image down inside the clip → exposes more of the top of the original photo** (useful when face is in the upper part of the source).
- The exact value depends on image aspect ratio and where the face sits. Just iterate.

## 8. Bird stories — full source list

See `birds_list.txt` for the canonical 1–52 list. Summary:
- Articles 1–10: already rendered in LaTeX.
- Articles 11–52: still in the original PDF/CDR, **not yet in the LaTeX book**.
- Three of the articles cover non-birds (Leopard and Cub, Cheetah, Lion — all Maasai Mara).
- Nilgiri Sholakili and Pallid Harrier each have two articles.

To add the remaining 42, the simplest approach is to extract the text from `Sample Layout English Final English.pdf` via `pdftotext`, then templated-render with the existing bird-page macros. The original build script `/tmp/build_latex.py` no longer exists; if a wholesale regenerate is needed, build the script fresh — the structure to follow is visible in the current `.tex` for birds 1–10.

## 9. Open items / decisions still to make

1. **Add the remaining 42 bird articles** to the LaTeX book.
2. **Source missing photos** for:
   - Common Green Magpie (article 4)
   - Common Emerald Dove (article 9)
   - Photos for any of articles 11–52 the user wants illustrated.
3. **Verify article 7's photo** (`frame 93.jpg`) — current image is likely Red-tailed Minla, not Red-billed Leiothrix as the article states. User accepted it, but a true Leiothrix photo would be ideal.
4. The cover-page treatment (`Untitled.png`) is full-bleed. If a real high-res cover photo is later supplied, drop it into the project root and update the path on line ~123 of the `.tex`.

## 10. User preferences observed

- **Coffee-table aesthetic:** big margins, generous whitespace, no clutter. Avoid overly-decorative typography.
- **Square portraits are the default** for attribution photos. The user explicitly switched p9 (daaj) and p17 (subra) to natural aspect, and later asked for p25 (skm) too — so when a portrait is a clean head-and-shoulders shot, ask whether square or natural is preferred.
- **The author's name is spelled "Daaji" (not "Daji").** Spelling correction applied throughout.
- **"Late" honorific prefix** is applied to deceased family members: Late Keshava Nayak, Late Madumathi Akka, Late Bharathi Pai, Late Janardhan Pai, Late Janardhana Pai.
- **Iteration is welcome.** The user prefers to see a quick first cut and then adjust. Don't over-engineer up front.
- **Italics for "starred" words:** the original DOCX used `*word*` syntax for emphasis. The LaTeX build script converts these to `\textbf{word}`. (Some pages were manually fixed during the session.)

## 11. Quick reference: file locations on this PC

```
/home/l1/gg/
├── Feather Fables - Life Essays.tex          # main LaTeX source
├── Feather Fables - Life Essays.pdf          # latest output (47 pages)
├── Feather Fables - Life Essays.tex.bak*     # backups
├── Sample Layout English Final English.pdf   # source of all bird article text
├── Sample Layout English Final.cdr           # original CorelDRAW (300 MB)
├── Untitled.png                              # cover image
├── birds_list.txt                            # 52-article list
├── PROJECT_MEMORY.md                         # this file
├── daaj.jpeg, twinsister.jpeg, anuradha.jpeg,
├── subra.jpeg, ullas_kamath.jpeg,
├── judge.jpeg, skm.jpeg, z.jpeg              # essay portraits
└── Vrathik2gb/Vrathik/                       # 251 bird/wildlife photos
```

## 12. When you (the next Claude) start

1. Read this file.
2. Glance at `Feather Fables - Life Essays.tex` (top 100 lines = preamble; the rest is content).
3. Compile once to verify the build works on the new machine.
4. Ask the user what they want to tackle next. The most likely next step is **either** adding more bird articles (11+) or refining a layout detail on an existing page.
