# Adopt a Reservist — print flyer

Source for `../adopt-a-reservist.pdf`: a single A4 page built as HTML and
printed to PDF with headless Chromium, using the same palette and typefaces
as the one-pager (Rubik / Assistant, `#333D36` green, `#E88225` orange,
`#FFFCF5` cream).

## Regenerating the PDF

```sh
cd flyer && python3 -m http.server 8778 &
node - <<'JS'
const { chromium } = require('playwright-core');
(async () => {
  const b = await chromium.launch();
  const p = await b.newPage();
  await p.goto('http://127.0.0.1:8778/adopt-a-reservist.html', { waitUntil: 'networkidle' });
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
- `assets/fonts.css` has the Rubik and Assistant latin subsets inlined as
  base64, so rendering needs no network and the PDF embeds the real faces.
- `assets/donate-qr.svg` is generated with `segno` from the donation URL.
  If the URL changes, regenerate it and re-decode it before printing.
