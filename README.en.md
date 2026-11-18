# makalah-universal

A tool that builds **Indonesian-language university papers (makalah)** as ready-to-submit `.docx` files, for many universities. The document format adapts to the user's university, faculty, and study program; when no match exists, a general Indonesian university format is used.

> **Made by Kurniawan Hafiz** | Version 5.0.0 | [Bahasa Indonesia](README.md)

## What's new in v5

- **Universal**: no longer tied to Information Systems of Universitas Riau. The format is picked automatically from the user's campus/program via `references/formats.json`, or forced with `meta.format`.
- **Default general format**: A4, margins 4-3-3-3 cm, Times New Roman 12, 1.5 spacing, justified, APA 7 reference list.
- **Individual papers**: `meta.jumlah_anggota = 1` produces a single-row "Nama Mahasiswa" cover. Groups keep "Nama Anggota 1..n".
- Margins, spacing, font size, and text width now follow the selected format instead of being hard-coded.
- Added `package.json`, `package-lock.json`, and `.gitignore`.

## Features

- Automatic cover, automatic table of contents with page numbers, heading numbering, Word equations, images, and an APA 7 reference list (hanging indent + justified).
- Content validator before build (`scripts/check_content.py`) and preview of key pages to PNG (`scripts/preview.sh`).
- One command for the whole pipeline: `python3 scripts/build.py content.json out.docx`.

See [CHANGELOG.md](CHANGELOG.md) for the full list.

## Supported formats

| id | University | Notes | Source |
|---|---|---|---|
| `default` | General Indonesia | A4, 4-3-3-3, TNR 12, 1.5 spacing | Compiled from various university guidelines |
| `unri:fmipa:si` | Universitas Riau | Information Systems, FMIPA | SI FMIPA UNRI thesis guidelines (2017) |
| `unri:fakultas-teknik` | Universitas Riau | Faculty of Engineering | FT UNRI thesis guide (2012) |
| `unri:fakultas-pertanian` | Universitas Riau | Faculty of Agriculture | FP UNRI thesis guidelines (2025) |
| `ugm:feb` | Universitas Gadjah Mada | Faculty of Economics and Business | FEB UGM academic writing guide |
| `ugm:kimia` | Universitas Gadjah Mada | Chemistry, FMIPA | Chemistry UGM proposal/thesis guide (2026) |

How the format is chosen:
1. `meta.format` in content.json (an id from the table above).
2. Automatically from `meta.universitas`, `meta.fakultas`, `meta.prodi`. Universitas Riau formats are preferred when the campus is UNRI.
3. No match = `default` format.

To add a new format, add an entry in `references/formats.json`. Field reference: [`references/schema.md`](references/schema.md).

## Usage with an AI assistant

Call the `/makalah-universal` skill and answer its questions: type (individual/group), university, faculty, study program, title, course, lecturer, topics, and sources. The AI will adapt the format to your program.

Example:

```
/makalah-universal
Write a paper.
- Type: group, 4 people
- University: Universitas Riau
- Faculty: Mathematics and Natural Sciences
- Program: Information Systems
- Title: Combinations, Permutations, and Binomial Coefficients
- Course: Discrete Mathematics
- Lecturer: Nama Dosen, gelar.
- Topics:
  - Definition and types of combinations
  - Definition and types of permutations
  - Binomial coefficients
- References: 10 journals, last 10 years, at least 5 in Indonesian
- Case study: 1, no references
```

After the file is generated:

- Edit member names, NIM (`( NIM )`), and the month-year in the preface.
- Right-click the table of contents and choose **Update Field** for exact page numbers, especially when the paper contains equations.

## Install (AI skill)

1. On the repo page, click **Code**, then **Download ZIP**.
2. In Claude/assis, open **Settings**, then **Skills**, and upload the zip.
3. Start a new chat so the skill is picked up.

Notes:

- If a skill with the same name already exists, delete it before uploading the new version.
- The ZIP from GitHub contains a folder named `makalah-universal-main`. If the upload is rejected because of the folder name, extract the zip, rename the folder to `makalah-universal`, re-zip, and upload again.

## Manual use (without an AI)

Requirements: Node.js, the `docx` npm package, Python 3, LibreOffice (`soffice`) for page counting, and optionally `pandoc` and `poppler-utils` (`pdftoppm`, `pdftotext`) for equation checks and preview.

```bash
npm install

# 1. copy and edit the example content (schema in references/schema.md)
cp assets/example_content.json content.json

# 2. one command: check, build, table of contents, validate, preview
python3 scripts/build.py content.json out/Paper.docx --tabel --preview
```

`build.py` options:

| Option | Function |
|---|---|
| `--strict` | Warnings stop the build |
| `--tabel` | Print the reference status table |
| `--preview` | Render key pages to PNG in `out/Paper_preview/` |
| `--tanpa-cek` | Skip the validator (not recommended) |

The `MAKALAH_SOFFICE` environment variable can point to a specific LibreOffice command.

## Folder structure

```
makalah-universal/
├── README.md                   # this file (Indonesian)
├── README.en.md                # English version
├── SKILL.md                    # AI skill instructions
├── CHANGELOG.md                # changelog
├── LICENSE                     # MIT
├── package.json                # npm metadata + docx dependency
├── package-lock.json
├── variant.json                # backward compatibility (unused since v5)
├── assets/
│   ├── logo_unri.png           # UNRI logo for covers
│   ├── example_content.json    # example paper content (equations, images, citations)
│   ├── example_diagram.png     # image for the example
│   └── prompt_template.md      # prompt template when the skill is not installed
├── references/
│   ├── schema.md               # content.json schema
│   └── formats.json            # document formats per campus/program
└── scripts/
    ├── build.py                # one command: check, build, TOC, validate, preview
    ├── check_content.py        # content validator before build
    ├── build_makalah.js        # builds .docx from content.json
    ├── omml.js                 # LaTeX to Word equation converter
    ├── finalize.py             # fills table of contents and page numbers
    └── preview.sh              # renders key pages to PNG
```

## `content.json` format

In short: `meta` (title, course, lecturer, university, year, member count, city, month-year, optional faculty, program, format, logo, max tables, ref_config), `kata_pengantar` (array of paragraphs), `bab` (array of chapters with `sections` and `blocks`), `daftar_pustaka` (array of APA 7 strings), `verifikasi_referensi`, and `sumber_dibuang`. Block types: `p`, `quote`, `list`, `table`, `equation`, `image`. Full reference: [`references/schema.md`](references/schema.md).

Important new fields:

- `meta.format`: force a format by id (e.g. `unri:fmipa:si`). Empty = automatic.
- `meta.jumlah_anggota`: `1` for individual papers, more than 1 for groups.

## Known limitations

- LibreOffice without the Math module cannot draw equations. On such machines equations appear blank in previews and table-of-contents page numbers are estimates. In Word they render normally; right-click the table of contents and choose **Update Field** for exact numbers.
- The equation converter supports a LaTeX subset (see schema). Unsupported commands fail the build loudly instead of silently producing wrong output.
- The validator reads citations of the form `Author (year)` and `(Author, year)`. Unusual patterns may be missed or misread (use `meta.lewati_sitasi`).
- Reference language is read from `verifikasi_referensi[].bahasa`; without it, only an estimate is used.
- The formats in `references/formats.json` are the guidelines currently available. If your campus/program is not listed, use the `default` format or send the official guideline to have it added.
- Always double-check every reference before submitting.

## Troubleshooting

| Symptom | Cause and solution |
|---|---|
| `Cannot find module 'docx'` | Run `npm install` in the skill folder, or install it globally. |
| `LibreOffice (soffice) tidak ditemukan` | Install LibreOffice or set `MAKALAH_SOFFICE`. |
| Cover without logo | The selected format has no logo; upload one via `meta.logo`. |
| Font/margins not as expected | Check `meta.format` and the `references/formats.json` entry; add a format if needed. |
| Build stops with citation errors | Citations in the text must exist in the reference list and vice versa; Chapter III and case studies must not cite. |
| Blank equations in preview | See limitations above. Verify in Word. |
| TOC page numbers off | Right-click the table of contents, **Update Field**. |
| ZIP rejected on skill upload | Rename the extracted folder to `makalah-universal`, re-zip. |

## Notes

- The UNRI logo belongs to Universitas Riau and is included only as a cover element for coursework.
- Member names, NIM, and dates are placeholders that must be edited by the user.
- Re-check every reference before submitting.

## License and credit

MIT licensed (see [LICENSE](LICENSE)). Made by **Kurniawan Hafiz**. Campus formats are compiled from each faculty/program's official guidelines (links in `references/formats.json`).
