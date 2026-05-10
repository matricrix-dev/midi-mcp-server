# AGENTS.md

## Dev Commands

```bash
npm run build       # Full build: clean -> build:ui (Vite) -> build:server (tsc)
npm run build:ui    # Vite builds MCP App preview HTML to dist/
npm run build:server # tsc + copies src/resources -> build/resources
npm test            # vitest run
npm run lint        # eslint
npm run format:check # prettier
npm run deploy      # wrangler deploy (Cloudflare Workers)
```

## Build Order Matters

`npm run build` runs: `clean` → `build:ui` → `build:server`
- `build:server` copies `src/resources/*.md` → `build/resources/` (music theory docs)
- `build:ui` outputs to `dist/` (NOT `build/`)
- `server.ts` loads the built HTML from `dist/src/mcp-app.html` at module level — if `build:ui` hasn't run, preview UI will show a fallback message
- CI order: `lint` → `format:check` → `test` → `test:coverage` → `build`

## Architecture

- Entry: `src/index.ts` — handles `--http`, `--port=N`, or stdio mode
- Server: `src/server.ts` — `createServer()` factory, registers MCP tools and resources
- `@tonejs/midi` is CJS-only; loaded via `createRequire` in `src/server.ts`
- Music theory resources live in `src/resources/*.md`, served via MCP resources

## tsconfig Quirk

`tsconfig.json` excludes `src/worker.ts`, `src/mcp-app.ts`, and `src/mcp-app.html` from compilation. These are for Cloudflare Workers and the MCP App preview, handled separately by Vite/Wrangler.
