# Adopt a Reservist — print flyers

Two A4 designs, both built as HTML and printed to PDF with headless Chromium.

| Source | Output | Design |
|---|---|---|
| `adopt-a-reservist.html` | `../adopt-a-reservist.pdf` | The website's system — Rubik / Assistant, `#333D36` green, `#E88225` orange, `#FFFCF5` cream. |
| `adopt-a-reservist-editorial.html` | `../adopt-a-reservist-editorial.pdf` | Editorial layout — blob-masked hero bleeding off the top-right, leaf sprigs, rounded sage pledge band. Same Rubik/Assistant and green/orange/cream as above. |

Both use the real workshop photograph, not a generated one.

## Regenerating the PDF

```sh
cd flyer && python3 -m http.server 8778 &
node - <<'JS'
const { chromium } = require('playwright-core');
(async () => {
  const b = await chromium.launch();
  const p = await b.newPage();
  await p.goto('http://127.0.0.1:8778/adopt-a-reservist.html', { waitUntil: 'networkidle' }); // or -editorial
  await p.evaluate(() => document.fonts.ready);
  await p.pdf({ path: '../adopt-a-reservist.pdf', printBackground: true, preferCSSPageSize: true });
  await b.close();
})();
JS
```

## Things that will break it

- **It must stay one page.** `.page` is a fixed 210x297mm box with
  `overflow: hidden`, so added copy is silently clipped rather than flowing
  to page 2. After any text change, check that `.page.scrollHeight` equals
  its rendered height (1123px at 96dpi) before shipping.
- **The donation URL is set in monospace on purpose.** It contains a `~`,
  which renders as a near-hyphen in Assistant at small sizes — anyone
  retyping it from print would land nowhere.
- `assets/fonts.css` has Rubik and Assistant inlined as base64, so rendering
  needs no network and the PDF embeds the real faces.
- **Both are fetched as STATIC woff via an old browser user-agent** (the v1
  `css?family=` endpoint, Chrome/40 UA). Google Fonts otherwise serves them
  as variable fonts, and Chromium's PDF export turns a variable font into
  Type3 glyph procedures — not a real embedded font, degraded at print
  resolution, and rejected by many commercial printers. This is silent:
  the PDF looks perfectly fine on screen. After any font change, audit the
  PDF for `/Type3` entries before sending anything to print.
- Neither Rubik nor Assistant ships an italic. Do not set `font-style:
  italic` — Chromium fakes an oblique and it prints badly. Use weight.
- `assets/donate-qr.svg` is generated with `segno` from the donation URL.
  If the URL changes, regenerate it and re-decode it before printing.
