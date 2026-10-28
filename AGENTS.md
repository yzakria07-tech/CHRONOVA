# Project notes

## What this is
A single static page: `index.html`, with Tailwind, GSAP, ScrollTrigger and Lenis
loaded from CDNs at runtime. No package manager, no build step, no backend,
no database, no environment variables or secrets.

The page's hero/lifestyle video is streamed from an external URL
(`www.designarena.ai`, which redirects to a Supabase signed URL). That host sends
no `Access-Control-Allow-Origin`, so the `<video>` tags must NOT carry
`crossorigin` — with it the media load fails outright and the hero goes blank.
The page never reads video pixels (no canvas/WebGL), so CORS mode is unnecessary.

## Running it locally (Base44 sandbox)
`docker-compose.base44.yml` runs `nginx:alpine` on host port 3000 with
`index.html` bind-mounted read-only into the web root, so edits show up on a
browser reload — nothing to rebuild or restart.

```sh
docker compose -f docker-compose.base44.yml up -d --build
curl -s localhost:3000 | head
```

Only `index.html` is mounted on purpose: mounting the repository root would serve
`.git/` over HTTP. If you add local assets (images, CSS, JS), mount that
directory (or the file) into `/usr/share/nginx/html/` as well and extend the
healthcheck below if it probes new content.

## Verifying
- Health: `docker compose -f docker-compose.base44.yml ps` should show `web` as
  `healthy`; the check greps the served page for `CHRONOVA`.
- Rendered check: open the preview on port 3000 — the fullscreen "CHRONOVA"
  hero with the watch, then the collection/spec sections on scroll.

## Deployment
`.github/workflows/static.yml` publishes the repository root to GitHub Pages on
pushes to `main`. That workflow, not this compose file, is the production path.
