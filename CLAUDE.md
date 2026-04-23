# percev.app

Public repo hosting the Percev **website**, **Sparkle appcast**, and **release binaries**. Served at https://percev.app via GitHub Pages (source: `main` branch, root).

## Repo layout

```
.
├── index.html        ← single-page landing (Percev brand, download CTA, features)
├── appcast.xml       ← Sparkle feed (RSS 2.0 + sparkle namespace). Rewritten by the release script.
├── CNAME             ← `percev.app` — GitHub Pages custom domain
└── CLAUDE.md         ← this file
```

Binaries live as GitHub Release assets (`Releases` tab) — they are NOT committed to the repo. The appcast's `<enclosure url=…>` points at `https://github.com/sunfmin/percev.app/releases/download/vX.Y.Z/Percev.dmg`.

## Where the source lives

The Percev macOS app source is in the **private** sibling repo `sunfmin/percev5` (locally cloned at `../percev5`). Do NOT put Swift/Xcode code here.

**Rule of thumb:**
- Website / landing-page content, colors, copy → edit here, commit, push → GitHub Pages rebuilds.
- Sparkle feed items → produced automatically by `../percev5/scripts/build-release.sh` during a release. Don't hand-edit `appcast.xml` unless you're rolling back a bad release.
- App code / features / tests → edit `../percev5/`.

## Release flow (recap)

The operator runs `../percev5/scripts/build-release.sh`. That script:
1. Bumps CFBundleVersion in `../percev5/Percev/Info.plist`.
2. Builds + signs (Developer ID) + notarizes + staples the DMG.
3. Signs the DMG with Sparkle's EdDSA key (private key in login Keychain).
4. Clones/pulls a scratch copy of this repo to `~/.percev-release/percev.app`, splices a new `<item>` into `appcast.xml`, commits + pushes to this repo's `main`.
5. Prints a ready-to-paste `gh release create vX.Y.Z … --repo sunfmin/percev.app …` for the operator to upload the DMG.

## DNS

- `percev.app` is registered at name.com.
- GitHub Pages apex setup: `percev.app` → CNAME `sunfmin.github.io.` (name.com supports CNAME flattening at apex). Alternative correct setup: four A records pointing to `185.199.108–111.153`.
- `CNAME` file in this repo must match the domain served — currently `percev.app`.
- HTTPS cert (Let's Encrypt) is auto-issued by GitHub after DNS is verified (10–60 min). If you see a `*.github.io` cert instead of `percev.app`, wait or unset-and-reset the custom domain in `Settings → Pages`.

## Local development

Plain HTML — no build step. Just open `index.html` in a browser. Or serve locally: `python3 -m http.server 8000`.

If you add a build step later (Astro, Next.js, etc.), output must land at repo root or `docs/` (and update Pages source accordingly).
