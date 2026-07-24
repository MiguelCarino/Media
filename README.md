# Media — Carino Systems

**[media.carino.systems](https://media.carino.systems)** — a client-side file & media manipulation suite. Every tool runs entirely in your browser: no uploads, no servers, no CDNs. All engines (pdf.js, Tesseract WASM, language models, fonts) are vendored in this repository, so the site works fully offline.

Consolidates and supersedes three earlier apps — **pdf2img-gui**, **SimpleOCR** and **SignatureEditor** — and adds two new tools.

## Tools

### 0. The file is the interface
There are no module tabs. The site opens on a single drop/paste surface (Ctrl+V works anywhere): the file is identified by its **magic bytes** — not its extension — and everything you can do with it is offered as a one-click outcome. Picking an outcome opens that workspace for the file; a **← File** button returns to the same detection result so you can run another task on it:

| Detected | Outcomes |
|---|---|
| PDF | convert pages to images · OCR via page render |
| PNG / JPEG / WebP / GIF / BMP / AVIF / SVG / **HEIC** | convert format & resize · OCR · signature cleanup |
| MP3 / FLAC / OGG / WAV / M4A | convert to WAV · ffmpeg command |
| MP4 / MOV / MKV / WebM | ffmpeg command · extract audio → WAV |
| anything else | first-bytes hex dump + a pointer to [metadata.carino.systems](https://metadata.carino.systems) |

### 1. PDF → Images
Render PDF pages to PNG, JPEG or WebP with [pdf.js](https://mozilla.github.io/pdf.js/) (vendored, v6).
- DPI presets (72 / 150 / 300) or any custom value from 36–600
- Page-range selection (`1, 3-5`), transparent or white background, quality slider for lossy formats
- Per-page download or a single ZIP of all pages

### 2. OCR — Text Recognition
Extract text from images with [Tesseract.js](https://tesseract.projectnaptha.com/) (vendored, v7 + core WASM v7), fully offline.
- Languages: **English, Español, Português** plus combined pairs — the `.traineddata.gz` models ship in `vendor/tessdata/` (Tesseract reads gzip natively)
- Paste (Ctrl+V), drag-drop or browse; rotate and invert preprocessing for white-on-dark text
- Confidence badge, copy to clipboard, `.txt` download

### 3. Signature Cleanup
Turn a photo/scan of a signature into a clean transparent PNG — pure canvas/JS, no OpenCV, and the whole UI **fits on one screen** (preview + thumbnail strip + one compact control column, no scrolling):
1. Grayscale (Rec.601 luma)
2. **Auto polarity** — the border ring decides whether ink is dark-on-light or light-on-dark (manual override available)
3. **Lighting flattening** — a coarse background estimate (area-average downscale → 3×3 max filter → box blur → bilinear upsample) divides out shadows and uneven paper, so a phone photo with a shadow across it no longer thresholds into a black slab
4. **Otsu automatic threshold** on the flattened image, with a 35 % ink-coverage cap as a safety net, plus a manual slider override
5. Background → transparent alpha, with optional soft (feathered) edges
6. **Despeckle** — connected-component analysis drops ink blobs under N px
7. Auto-crop to ink bounds with adjustable padding; ink recolor (black / blue / custom)
8. Manual eraser brush with undo; export transparent PNG at 1× / 2× / 4× (nearest-neighbour), single file or all-as-ZIP

### 4. Image Converter
Batch-convert images to PNG / JPEG / WebP via canvas re-encode.
- **HEIC/HEIF input** is decoded locally with a vendored [libheif](https://github.com/strukturag/libheif) WASM bundle (`vendor/libheif/`, loaded lazily on first HEIC) — iPhone photos convert without any upload
- Quality slider for lossy formats, optional max-dimension downscale (aspect preserved, never upscales)
- Per-file download or one ZIP for the whole batch

### 5. Transcode
Browsers can't transcode video offline without a ~30 MB WASM blob, so this is the honest tool:
- **ffmpeg command builder** — ports the intent of the old `transcode.sh`: H.264 / H.265 / AV1 / VP9, software or NVENC/QSV/AMF hardware encoders, CRF quality, resolution, audio/subtitle handling, two-pass for VP9/AV1 — and explains every flag it emits
- **Audio → WAV** — this one *does* run in-browser: anything your browser can decode (MP3, AAC, OGG, FLAC…) → 16-bit PCM WAV, with optional mono downmix

## Implementation notes

- **No frameworks, no build step, no runtime network calls.** Vanilla JS in a single `index.html`.
- The ZIP downloads use a ~60-line **store-only ZIP writer** written for this project (local file headers + central directory + CRC-32) — no JSZip.
- File saving is a plain blob + anchor helper — no FileSaver.js.
- `vendor/pdfjs/` — pdf.js build (`pdf.min.mjs` + worker), dynamically imported on first use.
- `vendor/tesseract/` — tesseract.js `tesseract.min.js` + `worker.min.js` and all `tesseract-core-*.wasm.js` variants (plain / SIMD / relaxed-SIMD, each with an LSTM-only build); the worker picks the best one for the CPU at runtime.
- `vendor/tessdata/` — `eng` / `spa` / `por` traineddata, kept gzipped.
- `vendor/libheif/` — libheif WASM single-file bundle (wasm inlined as base64), injected only when a HEIC file is actually opened; HEIC is normalized to PNG at the door so OCR, Signature and the converter all accept it.
- The page is a **fixed-viewport layout**: the body never scrolls — long content scrolls inside its own panel.
- Fonts are self-hosted (`fonts/carino-fonts.css`); the shared fleet navbar is `carino-navbar.js` + `carino-clock.js`.
- Internals are exposed on `window.MediaTools` for testing.

## License

This project is licensed under the **GNU Affero General Public License v3.0** — see [LICENSE](LICENSE).

Vendored components keep their own licenses: pdf.js (Apache-2.0, `vendor/pdfjs/LICENSE`), tesseract.js (Apache-2.0, `vendor/tesseract/LICENSE.md`), tesseract.js-core (Apache-2.0, `vendor/tesseract/LICENSE-core`), tessdata language models (Apache-2.0), libheif (LGPL-3.0, `vendor/libheif/LICENSE`).

---
Part of the [Carino Systems](https://carino.systems) fleet.
