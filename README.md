# Media — Carino Systems

**[media.carino.systems](https://media.carino.systems)** — a client-side file & media manipulation suite. Every tool runs entirely in your browser: no uploads, no servers, no CDNs. All engines (pdf.js, Tesseract WASM, language models, fonts) are vendored in this repository, so the site works fully offline.

Consolidates and supersedes three earlier apps — **pdf2img-gui**, **SimpleOCR** and **SignatureEditor** — and adds two new tools.

## Tools

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
Turn a photo/scan of a signature into a clean transparent PNG — the OpenCV pipeline of the old SignatureEditor reimplemented in ~200 lines of pure canvas/JS (no 9 MB WASM download):
1. Grayscale (Rec.601 luma)
2. **Otsu automatic threshold** with manual slider override, invert mode for light-ink-on-dark
3. Background → transparent alpha, with optional soft (feathered) edges
4. **Despeckle** — connected-component analysis drops ink blobs under N px
5. Auto-crop to ink bounds with adjustable padding
6. Ink recolor: black / blue / custom color
7. Manual eraser brush with undo, pipeline-step debug strip, per-step export
8. Export transparent PNG at 1× / 2× / 4× (nearest-neighbour, crisp edges)

### 4. Image Converter
Batch-convert images between PNG / JPEG / WebP via canvas re-encode.
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
- Fonts are self-hosted (`fonts/carino-fonts.css`); the shared fleet navbar is `carino-navbar.js` + `carino-clock.js`.
- Internals are exposed on `window.MediaTools` for testing.

## License

This project is licensed under the **GNU Affero General Public License v3.0** — see [LICENSE](LICENSE).

Vendored components keep their own licenses: pdf.js (Apache-2.0, `vendor/pdfjs/LICENSE`), tesseract.js (Apache-2.0, `vendor/tesseract/LICENSE.md`), tesseract.js-core (Apache-2.0, `vendor/tesseract/LICENSE-core`), tessdata language models (Apache-2.0).

---
Part of the [Carino Systems](https://carino.systems) fleet.
