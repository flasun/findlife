# OP-COM // Operational Command

Tactical executive coach. BLUF first. Wisdom Council on demand. Powered by Claude Opus 4.7.

## Quickstart

```bash
cp .env.example .env.local
# add your ANTHROPIC_API_KEY
pnpm install
pnpm dev
```

Open http://localhost:3000.

## Stack

- Next.js 15 (App Router) + React 19 + TypeScript
- Tailwind CSS v4
- `@anthropic-ai/sdk` calling `claude-opus-4-7` directly
- Streaming via `messages.stream()`, `cache_control: ephemeral` on the system block
- Conversation persists in `localStorage` (no server-side storage in v0)

## Deploy (Cloudflare Workers)

The app runs on Cloudflare Workers through the [OpenNext adapter](https://opennext.js.org/cloudflare), as the Worker `findlife` at https://findlife.racason.workers.dev.

1. Store the API key as a Worker secret (once): `pnpm exec wrangler secret put ANTHROPIC_API_KEY`
2. Deploy, either way:
   - From your machine: `pnpm deploy:cf` (builds with `opennextjs-cloudflare build`, then runs `wrangler deploy`; run `pnpm exec wrangler login` first).
   - From GitHub: add the repo secrets `CLOUDFLARE_API_TOKEN` and `CLOUDFLARE_ACCOUNT_ID`, then run the **Deploy to Cloudflare** workflow from the Actions tab.

`pnpm preview:cf` builds the Worker and serves it locally with `wrangler dev` (local secrets go in `.dev.vars`, e.g. `ANTHROPIC_API_KEY=...`). `pnpm dev` is still the fastest loop for UI work.

## Caching note

Anthropic's prompt cache requires a minimum prefix of 4096 tokens on Opus 4.7. The OP-COM system prompt is currently below that threshold, so cache reads will be 0 until the prompt is expanded (e.g., with examples, mentor playbooks, or memory context). The `cache_control` directive is already wired — no code change needed when the prefix grows past the minimum.
