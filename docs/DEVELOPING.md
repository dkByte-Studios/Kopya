# Developing Kopya

## Put it on GitHub Pages

1. Create a new repository on GitHub, for example `kopya`.
2. Upload everything in this folder to the repository (on the repo page: **Add file → Upload files**, then drag in all the files and folders, keeping the folder structure).
3. Go to **Settings → Pages**. Under **Build and deployment**, set **Source** to *Deploy from a branch*, choose the `main` branch and the `/ (root)` folder, then **Save**.
4. After a minute or two the site is live at `https://YOUR-USERNAME.github.io/kopya/`.

The `.nojekyll` file tells GitHub Pages to serve the files exactly as they are.

You can link straight to one tool by adding its name after `#`, for example `https://YOUR-USERNAME.github.io/kopya/#pdf-to-word`.

## Try it on your computer

Browsers block the converters when the page is opened as a plain file, so serve the folder instead:

```
python3 -m http.server 8000
```

Then open `http://localhost:8000`.

## How it's built

There's no build step: plain HTML, CSS and JavaScript modules.

```
index.html          the page
css/style.css       styles (light and dark)
js/app.js           tool list, file queue, downloads
js/converters/      one file per conversion, loaded only when used
js/lib/             shared code: PDF text reading, page layout, Word and PowerPoint renderers, XLSX and EPUB writers, image reading and resizing
vendor/             the open-source libraries Kopya uses (PDF.js, pdf-lib, docx, JSZip, html-to-image, lamejs, libheif, the AVIF encoder)
docs/screenshots/   the images used in the README
```

The libraries are included in the repository rather than loaded from a CDN, so the site keeps working even if a CDN is down. The large ones (the AVIF encoder is 3.5 MB, libheif 1.4 MB) are fetched only when you use them. Their licenses are in `vendor/licenses/THIRD-PARTY-NOTICES.txt`.

## Conversion notes

| Conversion | How well it works |
|---|---|
| PDF → JPG | Pixel-perfect page images. Several pages download as a ZIP. |
| PDF → Word (.docx) | Editable text with headings, bold and italics. Scanned pages come through as pictures. Pictures inside a text PDF aren't carried over. "Exact look" mode places each page as a picture. |
| PDF → Excel (.xlsx) | Rows and columns rebuilt from the text's position. Best on simple tables. |
| PDF → EPUB | Reflowable e-book with chapters taken from the headings. Running headers, footers and page numbers are removed. |
| Word (.docx) → PDF | Styles, lists, tables, images, headers, footers and page numbers. Fonts that aren't installed are swapped for similar ones. |
| PowerPoint (.pptx) → PDF | Themes, layouts, text, shapes, pictures, tables, groups and SmartArt. Charts aren't drawn yet. |
| EPUB → PDF | A5, A4 or Letter pages with page numbers and the cover. DRM-protected books can't be converted. |

Images: JPG, PNG, WebP and BMP are drawn and saved by the browser. SVG is drawn at the size you pick (at least 1024 px on the long side), and scripts or outside links inside an SVG are never run. Saving AVIF uses a WebAssembly encoder in its own worker, loaded only when AVIF is picked. HEIC is decoded with libheif, loaded only when a HEIC file is added. Pictures over about 64 megapixels (36 for AVIF) are shrunk automatically.

Audio: MP3 bitrates are 128, 192, 256 and 320 kbps, saved at 44.1 kHz; surround is mixed down to stereo. Files over 150 MB or 3 hours are refused because the browser holds the whole decoded track in memory. Which formats can be opened depends on the browser (Safari can't open OGG).
