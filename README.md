# Just A Second — One Pager (English)

English (LTR) version of the Just A Second one-pager.

- **Live:** https://jas-one-pager-donation.netlify.app
- **Hebrew original:** https://jas-one-pager.vercel.app — source at
  [Yuval-Steimberg/JAS-One-Pager](https://github.com/Yuval-Steimberg/JAS-One-Pager)

## Structure

A single static `index.html` with inline CSS and one small IntersectionObserver
script for scroll reveals. Images sit next to it at the repo root. No build step.

Preview locally:

```sh
python3 -m http.server 8000
```

## Keeping the two versions in sync

The DOM structure of this `index.html` is identical to the Hebrew one — only the
text content, `lang`/`dir`, the metadata block and one direction-dependent border
differ. When the Hebrew page changes, apply the same structural change here, or
the two will drift.
