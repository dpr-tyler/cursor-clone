# Polaris

Polaris is a web app for **AI-assisted coding projects**: create workspaces from a prompt, chat with an agent that reads and edits your file tree, and **import or export repositories** via GitHub (using your connected GitHub account through Clerk).

## Stack

- **Framework:** [Next.js 16](https://nextjs.org) (App Router), React 19, TypeScript
- **UI:** Tailwind CSS 4, [shadcn](https://ui.shadcn.com)-style components, [Base UI](https://base-ui.com) / Radix primitives
- **Data:** [Convex](https://www.convex.dev) — projects, virtual file tree, conversations, and chat messages
- **Auth:** [Clerk](https://clerk.com) with [Convex + Clerk](https://docs.convex.dev/auth/clerk) integration
- **Background jobs:** [Inngest](https://www.inngest.com) — assistant message pipeline, GitHub import/export
- **Models:** Anthropic Claude via [Inngest Agent Kit](https://www.inngest.com/docs/features/agents) and the [Vercel AI SDK](https://sdk.vercel.ai/docs)
- **Scraping:** [Firecrawl](https://firecrawl.dev) for URL context inside agent tools
- **Observability:** [Sentry](https://sentry.io) (`@sentry/nextjs`), including Vercel AI instrumentation on the server

The dev server sends **Cross-Origin Embedder Policy** headers (`credentialless` / `same-origin` opener) to support browser features such as [WebContainers](https://webcontainers.io).

## Repository layout

| Path | Purpose |
|------|---------|
| `src/app/` | App Router pages, layouts, and `api/` route handlers |
| `src/features/` | Domain UI and logic (`projects`, `conversations`, `auth`, …) |
| `src/components/` | Shared UI (design system, providers) |
| `convex/` | Convex schema, auth config, and server functions |

## Prerequisites

- **Node.js** (LTS recommended)
- **npm** (this repo uses `package-lock.json`)

## Setup

### 1. Install dependencies

```bash
npm install
```

### 2. Convex

Link or create a Convex project and keep codegen running while you develop:

```bash
npx convex dev
```

This updates `convex/_generated/*` and deploys functions to your dev deployment.

### 3. Environment variables

Add a `.env.local` in the project root for Next.js. You must also define the same Convex-related secrets in the **Convex dashboard** (or via `npx convex env set`) where the backend reads them — for example `POLARIS_CONVEX_INTERNAL_KEY` and `CLERK_JWT_ISSUER_DOMAIN`.

| Variable | Where it’s used | Notes |
|----------|-----------------|--------|
| `NEXT_PUBLIC_CONVEX_URL` | Client + server | Convex deployment URL |
| `POLARIS_CONVEX_INTERNAL_KEY` | Next.js API routes, Inngest steps, Convex `system` functions | Shared secret; must match between Next.js, Inngest environment, and Convex |
| `CLERK_JWT_ISSUER_DOMAIN` | Convex (`convex/auth.config.ts`) | Clerk JWT issuer domain for Convex auth |
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Clerk (Next.js) | From Clerk dashboard |
| `CLERK_SECRET_KEY` | Clerk (Next.js) | From Clerk dashboard |
| `ANTHROPIC_API_KEY` | AI / agent runtime | Required for Claude |
| `FIRECRAWL_API_KEY` | Firecrawl client | Used by scraping-related tools |

**Inngest:** configure your app URL and signing credentials for `/api/inngest` per the [Inngest Next.js quick start](https://www.inngest.com/docs/getting-started/nextjs-quick-start) so background functions (`process-message`, GitHub import/export) can run.

**GitHub:** repo import uses the user’s **GitHub OAuth token from Clerk** (and the import API checks for a **Pro** plan). Export uses the same connected account.

### 4. Run the app

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000). You should be signed in via Clerk to use the main experience.

## Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Next.js development server |
| `npm run build` | Production build |
| `npm run start` | Start production server |
| `npm run lint` | ESLint |

## Deploy

The app is set up for deployment on [Vercel](https://vercel.com) (Next.js). Set the same environment variables in Vercel, Convex, and Inngest for production, and point Inngest at your production `/api/inngest` URL.
