# Adopt a Reservist — print flyers

Two A4 designs, both built as HTML and printed to PDF with headless Chromium.

| Source | Output | Design |
|---|---|---|
| `adopt-a-reservist.html` | `../adopt-a-reservist.pdf` | The website's system — Rubik / Assistant, `#333D36` green, `#E88225` orange, `#FFFCF5` cream. |
| `adopt-a-reservist-editorial.html` | `../adopt-a-reservist-editorial.pdf` | Editorial — Playfair Display / Lato, `#234432` forest, `#C06B47` terracotta, blob-masked hero, sage pledge band. |

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
- `assets/fonts.css` (Rubik/Assistant) and `assets/fonts-editorial.css`
  (Playfair Display/Lato) have their latin subsets inlined as base64, so
  rendering needs no network and the PDF embeds the real faces.
- **Playfair Display is fetched as a STATIC woff, via an old browser
  user-agent.** Google Fonts otherwise serves it as a variable font, and
  Chromium's PDF export turns variable fonts into Type3 glyph procedures:
  not a real embedded font, degraded at print resolution, and rejected by
  many commercial printers. After changing fonts, confirm no `/Type3`
  entries survive in the PDF before sending anything to print.
- `assets/donate-qr.svg` is generated with `segno` from the donation URL.
  If the URL changes, regenerate it and re-decode it before printing.
