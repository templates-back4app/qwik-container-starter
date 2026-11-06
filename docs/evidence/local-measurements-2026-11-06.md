# Local measurements — 2026-11-06

Machine: Apple M5, macOS 26.5.2. Docker: colima VM (docker 29.5.2, linux/aarch64, 4 CPUs, 8 GB) — Docker Desktop was not installed on the work machine, so the docker CLI + colima were installed via Homebrew.
Node v22.23.4 / npm 10.9.9 (host builds; images use `node:22-alpine`).

Packages: @builder.io/qwik@1.20.3, @builder.io/qwik-city@1.20.3, express@4.22.0, compression@1.8.2, vite@7.3.1, undici@7.30.2, dotenv@16.6.1.

| Metric | Value | Command |
|---|---|---|
| npm ci | 2.9 s | `rm -rf node_modules && time npm ci` |
| npm run build | 6.3–6.9 s | `time npm run build` (client + SSR server + type-check + lint) |
| Output folders | 188 KB (`dist/`) + 148 KB (`server/`) | `du -sh dist server` |
| docker build (no cache) | 56.5–56.7 s | `time docker build --no-cache -t qwik-container-starter .` |
| Image size | 249 MB | `docker images --format '{{.Size}}'` |
| First-load JS (gzip) | 23,846 B (~23.3 KB) across 3 preloaded files | see method below |

First-load JS method: `docker run -p 8091:8080 …`, `curl http://localhost:8091/`, collect every `.js` URL referenced from the page (script `src` + `rel="modulepreload"` links), then curl each with `Accept-Encoding: gzip` and sum `size_download`. Caveat to be fair to Qwik: these three chunks are *preloaded* by the page; Qwik's resumability means they are not executed until interaction, and the only JS executed on load is the small inline qwikloader.

Notes / pitfalls:
- `npm create qwik@latest` has no "basic" template: the choices are `empty` (the standard starter app with routing — used here), `playground`, `e2e-library`, `library`. Non-interactive scaffold: `npm create qwik@latest empty ./qwik-container-starter -- --installDeps`.
- `npm run qwik add express` is interactive and dies without a TTY (`ERR_TTY_INIT_FAILED`). Non-interactive flag (undocumented, from the CLI source): `npx qwik add express --skipConfirmation=true`.
- The generated `src/entry.express.tsx` ships with `app.use(compression())` **commented out** — without it every asset is served raw (the main chunk measured 49.9 KB instead of 20.2 KB). We enabled it.
- The built `server/entry.express.js` imports `express`, `dotenv/config` and `undici` at runtime, but the scaffold puts `dotenv`, `undici` and `compression` in devDependencies — `npm ci --omit=dev` in the runtime image then crashes. We moved all three to `dependencies`.
- The Express server listens on `PORT` (default 3000) and binds all interfaces (Express default), so no `HOST` fiddling is needed.
