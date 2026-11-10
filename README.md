# FixKit

A Windows desktop app that helps beginners troubleshoot Visual Studio Code and C++ toolchains (MinGW-w64, MSYS2, GCC/G++, GDB). Electron + React + TypeScript + Tailwind, a local SQLite store, a local JSON knowledge base, and OpenRouter for AI diagnosis and screenshot reading.

## Run

```bash
npm install
npm run dev      # Vite + Electron with hot reload
npm start        # production build, then launch
npm test         # knowledge-base retrieval tests
npm run dist     # Windows installer in release/
```

Add an OpenRouter key in **Settings**, or set `OPENROUTER_API_KEY` for development. With no key the app runs offline and answers only from the knowledge base.

> **Behind a TLS-inspecting network?** If installing Electron fails with `self-signed certificate in certificate chain`, run `NODE_OPTIONS=--use-system-ca node node_modules/electron/install.js` so Node trusts the Windows certificate store.

The installer is unsigned (`signAndEditExecutable: false`), so Windows SmartScreen will warn on first run and the exe has the default Electron icon. To sign it, add a code-signing certificate and remove that flag. `electronDist` packages the Electron already in `node_modules`, so the build doesn't download it again.

## Architecture

| Path | Role |
| --- | --- |
| `electron/main.ts` | Window, IPC handlers, screenshot storage |
| `electron/engine.ts` | Troubleshooting engine: retrieval → match kind → prompt → structured diagnosis; offline fallback |
| `electron/kb.ts` | Knowledge-base loader, hybrid retrieval (error-pattern + phrase + BM25), category classification, scope detection |
| `electron/openrouter.ts` | OpenRouter client with timeout, auth, rate-limit, credit and model error mapping |
| `electron/db.ts` | SQLite (sql.js) with ordered migrations: sessions, messages, preferences |
| `electron/secrets.ts` | API key encrypted with Electron `safeStorage` (Windows DPAPI), stored outside the DB |
| `electron/shared.ts` | Types shared by main and renderer |
| `src/` | React UI: dashboard, workspace, knowledge base, history, settings, about |
| `knowledge/core.json` | 64 bundled troubleshooting records with official documentation links |

The renderer runs sandboxed with context isolation, and the API key never reaches it. The app never runs commands on the user's machine.

## Extending the knowledge base

Put `.json` files (an array of records, or `{ "records": [...] }`) in the user knowledge folder (**Settings → Open user knowledge folder**), then press reload in the Knowledge base screen. A record with an existing `id` replaces the bundled one. See `electron/shared.ts` (`KbRecord`) for the schema.
