# Qwik Container Starter

Qwik + Qwik City starter (`npm create qwik@latest`, empty app) with the **Express/Node adapter** (`npx qwik add express`): `npm run build` produces the client in `dist/` and a production Express server in `server/entry.express.js` that listens on `PORT` (default 3000; the Dockerfile sets 8080). Gzip compression is enabled in the Express entry.

Run locally with Docker:

```sh
docker build -t qwik-starter .
docker run -p 8080:8080 qwik-starter
# open http://localhost:8080
```

Deploy guide: see the Back4app blog post for deploying this repo on Back4app Containers (placeholder — link coming with the tutorial).

Measurements for this exact commit are in `docs/evidence/local-measurements-2026-11-06.md`.
