# Personal Portfolio

Personal portfolio for **MD. Omar Farook Hridoy** — ERP Technical Lead & Fintech Software Engineer.

**Live site:** https://farookhridoy.com

## Overview

A fast, static, single-page portfolio built with the Next.js App Router. It presents a resume, selected work, open-source repositories, writing links, and contact channels in a responsive shell with dark mode and subtle motion.

- One user-facing route (`/`). Resume / Work / Posts tabs and the Contact view are hash-driven client state, not separate pages.
- Content is typed, hard-coded data in `lib/content.ts`.
- The GitHub contribution heatmap uses a generated snapshot in `lib/contributions.json`.
- Public GitHub repositories are fetched in the browser from the unauthenticated GitHub API.
- `output: 'export'` produces a fully static `out/` directory that can be hosted anywhere.

## Tech stack

| Area | Choice |
| --- | --- |
| Framework | Next.js 15 (App Router), React 18 |
| Language | TypeScript, strict config with `@/*` path alias |
| Styling | Tailwind CSS 3, CSS-variable design tokens, `next-themes` |
| Motion | Framer Motion |
| UI | shadcn/ui + Radix primitives, Lucide icons, Simple Icons |
| SEO | App Router metadata, JSON-LD, `app/robots.ts`, `app/sitemap.ts`, `next-sitemap` |
| Deployment | Static export to `out/` |

## Getting started

Requirements: Node.js 18.18+ (Node 20+ recommended) and npm.

```bash
git clone https://github.com/farookhridoy/farookhridoy-portfolio.git
cd farookhridoy-portfolio
npm install
npm run dev
```

Open http://localhost:3000.

### Scripts

| Command | Description |
| --- | --- |
| `npm run dev` | Start the development server |
| `npm run build` | Create the static production export in `out/` (also runs `postbuild`) |
| `npm run postbuild` | Run `next-sitemap` for SEO files |
| `npm run lint` | Run `next lint` (deprecated in installed Next 15) |
| `npm run start` | Start the Next.js server; not the normal path for the static export |

Useful manual checks:

```bash
npx tsc --noEmit
npx eslint .
```

`next.config.ts` currently ignores TypeScript and ESLint errors during `next build`, so the two commands above are the real correctness checks. There is no test suite yet.

## Project structure

```text
.
├── app/
│   ├── globals.css        # Tailwind layers, light/dark tokens, base styles
│   ├── layout.tsx         # Root layout, metadata, fonts, providers, toasters
│   ├── page.tsx           # Client shell: sidebar, tabs, contact view
│   ├── providers.tsx      # QueryClientProvider
│   ├── robots.ts          # Static /robots.txt
│   └── sitemap.ts         # Static /sitemap.xml
├── components/
│   ├── seo/               # JSON-LD structured data
│   ├── shell/             # Portfolio-specific UI components
│   ├── ui/                # Generated shadcn/ui primitives
│   └── theme-provider.tsx
├── hooks/                 # GitHub repos, mobile breakpoint, toast store
├── lib/
│   ├── content.ts         # All portfolio copy and data
│   ├── contributions.json # Generated GitHub contribution snapshot
│   ├── tool-icons.ts      # Brand glyph mapping
│   └── utils.ts           # cn() helper
├── public/
│   ├── images/            # Product and project screenshots
│   └── favicon.png, placeholder.svg
├── scripts/
│   └── fetch-contributions.mjs
├── AGENTS.md              # Development guide for humans and AI agents
├── PROJECT_ANALYSIS.md    # Detailed architecture analysis
├── DEPLOYMENT.md          # Static-hosting instructions
└── next.config.ts         # Static export configuration
```

Generated and disposable: `.next/`, `out/`, `next-env.d.ts`, `tsconfig.tsbuildinfo`.

## Editing the site

### Content

Most copy and data live in `lib/content.ts`:

- Profile, bio, stats, skills and social links
- Experience and education
- Products, live projects and legacy projects
- LinkedIn posts, work-with-me cards and repository fallbacks

Text wrapped in `**double asterisks**` is rendered as emphasis by `components/shell/Rich.tsx`.

When changing the name, title or description, also update:

- `app/layout.tsx` metadata
- `components/seo/StructuredData.tsx` JSON-LD

### Images

Place images in `public/images/`:

- `public/images/products/` for product cards
- `public/images/projects/` for live project cards

Reference them with root-relative paths such as `/images/projects/tavrix.jpg`.

### GitHub repositories

`hooks/use-github-repos.ts` fetches public repos for `profile.githubUser` in `lib/content.ts`.

- Hide a repo: add its name to `hiddenRepos` in `lib/content.ts`.
- Provide a fallback description: add an entry to `repoFallbackDescriptions`.
- The API call is unauthenticated, so rate limits can affect the Open source section.

### Contribution heatmap

`lib/contributions.json` is a snapshot, not live data. Regenerate it with a signed-in `gh` CLI:

```bash
node scripts/fetch-contributions.mjs farookhridoy
npm run build
```

### Theme and branding

- Colors, radius and base styles: `app/globals.css`
- Tailwind color mapping and plugins: `tailwind.config.ts`
- Fonts: `app/layout.tsx`

## Deployment

Build the site:

```bash
npm run build
```

Then upload the **contents** of `out/` to any static host. See [`DEPLOYMENT.md`](./DEPLOYMENT.md) for cPanel/FTP, Netlify, Vercel and GitHub Pages instructions.

Notes:

- No Node.js runtime is needed on the host.
- `SITE_URL` can be set for `next-sitemap`; it defaults to `https://farookhridoy.com`.
- GitHub Pages under a subpath (for example `username.github.io/repo-name`) requires `basePath`/asset-path changes because current URLs are absolute.
- Tracked `public/robots.txt` and `public/sitemap.xml` may be refreshed by `postbuild`; the canonical static-export versions are generated into `out/`.

## Further reading

- [`AGENTS.md`](./AGENTS.md) — development playbooks, conventions and gotchas
- [`PROJECT_ANALYSIS.md`](./PROJECT_ANALYSIS.md) — detailed architecture analysis
- [`DEPLOYMENT.md`](./DEPLOYMENT.md) — deployment guide
