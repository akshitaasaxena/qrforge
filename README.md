# QRForge

Turn any link into a QR code — generate, customize, and download a scannable
code in seconds. Everything runs client-side: no account, no backend, no
link ever leaves the browser.

## Getting started

Requires Node.js 18+.

```bash
npm install
npm run dev       # start the dev server (http://localhost:5173)
npm run build     # production build, output in dist/
npm run preview   # preview the production build locally
npm run lint      # oxlint
```

## What's here (MVP)

- Paste a URL -> validated -> encoded into a QR code, rendered as both SVG
  (crisp on-screen preview) and PNG (instant download).
- Customization: foreground/background color (with a live contrast warning),
  size, margin (quiet zone), and error correction level.
- Download as PNG or SVG, copy the destination URL, and share via the Web
  Share API where the browser supports it.
- Pages: `/`, `/about`, `/help`, `/privacy`, `/terms`.
- Generated codes were round-trip tested with a real QR decoder (`jsQR`)
  across several cases -- short/long URLs, query strings + fragments,
  custom colors, and minimum margin -- to confirm they actually scan, not
  just render.

## Architecture, briefly

```
src/
├── components/   UI only -- rendering, interaction, local state
├── hooks/        reusable stateful behavior (useQRCode, useClipboard, useLocalStorage)
├── utils/        pure functions (validation, normalization, encoding, contrast)
├── services/     application operations (qrService.generate, the QR value provider)
├── constants/    defaults and option sets, in one place
└── pages/        route-level components
```

Two seams are worth knowing about if you're extending this:

- **`src/services/qrValueProvider.js`** decouples "the string encoded into
  the QR" from "the URL the user typed." Today `StaticQRValueProvider` just
  returns the destination URL as-is. A future `DynamicQRValueProvider` can
  mint a short redirect URL (`https://qrforge.app/r/abc123`) backed by a
  server, and nothing above this seam -- not `qrService`, not any component --
  needs to change.
- **`src/services/qrService.js`** is the one place UI code calls into for
  "generate a QR code." If dynamic QR, saved history, or analytics need a
  backend call later, it's added here, not scattered across components.

The project intentionally ships without a backend, auth, or a database --
see the original product brief for the full roadmap (dynamic QR, scan
analytics, history, accounts). The folder structure and these two seams
exist so those can be layered on without restructuring what's here.

## Testing

Core logic (`urlValidator`, `urlNormalizer`, `qrGenerator`) is pure and has
no DOM dependency, so it's straightforward to unit test with any runner
(Vitest is a natural fit given the Vite setup, not included by default).

To re-verify actual scannability of generated codes, the QR output was
decoded with jsQR (https://www.npmjs.com/package/jsqr) against a PNG
produced by the same `qrcode` options this app uses, confirming the decoded
string exactly matches the encoded URL across multiple sizes, error
correction levels, and color combinations.
