# Third-party code bundled here

Everything this suite needs is vendored: it loads no script, style, font, map or
module from anywhere but its own origin — no CDN, no network, no account. That
is the point of the tool (a PDF, an image or an audio file never leaves the
machine), and it makes the licences below *this project's* responsibility rather
than a package manager's.

**Every licence text is present in this tree.** Each row points at the file.

## What is here, and under what licence

| Path | Package | Licence | Text |
| --- | --- | --- | --- |
| `pdfjs/pdf.min.mjs`, `pdf.worker.min.mjs` | [PDF.js](https://mozilla.github.io/pdf.js/) | Apache-2.0 | `pdfjs/LICENSE` |
| `tesseract/tesseract.min.js`, `worker.min.js` | [Tesseract.js](https://github.com/naptha/tesseract.js) | Apache-2.0 | `tesseract/LICENSE.md` |
| `tesseract/tesseract-core*.wasm.js` | [Tesseract OCR](https://github.com/tesseract-ocr/tesseract) engine, compiled to WASM | Apache-2.0 | `tesseract/LICENSE-core` |
| `tessdata/*.traineddata.gz` | [tessdata](https://github.com/tesseract-ocr/tessdata) models (eng, spa, por) | Apache-2.0 | covered by `tesseract/LICENSE-core` |
| `libheif/libheif-bundle.js` | [libheif](https://github.com/strukturag/libheif) + libde265, compiled to WASM | **LGPL-3.0** | `libheif/LICENSE` |

## The one entry that needs thought before you redistribute

**`libheif` is LGPL-3.0**, and its bundle carries `libde265` with it. LGPL-3.0 is
compatible with this project's AGPL-3.0, so there is no conflict — but the LGPL
asks that a user be able to replace the library. For a WebAssembly bundle that
means shipping it unmodified (as here) and saying so (this file).

If a build of `libheif` that includes an **x265 encoder** is ever substituted,
that component is GPL-licensed and the analysis changes. Check before swapping
the bundle.

## The rule

Adding a file to this directory means adding a row here, and its licence text
beside it, **in the same commit**. A record kept from the first vendored file is
trivial; one reconstructed two years later is not.
