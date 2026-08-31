# ankitp.com.np

Personal site and blog. Hugo + [Blowfish](https://blowfish.page) theme, loaded as a Hugo module.

## Develop

```sh
nix develop      # shell with hugo + go
hugo server      # http://localhost:1313
```

## Update the theme

```sh
hugo mod get -u github.com/nunocoracao/blowfish/v2
hugo mod tidy
```

## Deploy

Push to `master`; GitHub Actions (`.github/workflows/gh-pages.yml`) builds and deploys to GitHub Pages.

## Layout notes

- Config lives in `config/_default/` (Blowfish conventions).
- The only theme override is `layouts/partials/comments.html` (giscus) — Blowfish's documented extension point.
- Posts are page bundles under `content/posts/`; a cover image is any bundle file matching `*feature*`, `*cover*`, or `*thumbnail*`.
- Old `/p/<slug>/` URLs redirect via per-post `aliases`.
