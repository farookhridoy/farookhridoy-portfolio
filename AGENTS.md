# AGENTS.md — terminal-dev-portfolio

Working guide for humans and AI agents in this repository. A deeper write-up is in [`PROJECT_ANALYSIS.md`](./PROJECT_ANALYSIS.md).

## 1. Project snapshot

- Personal portfolio for **MD. Omar Farook Hridoy** — "ERP Technical Lead & Fintech Software Engineer".
- **One App Router page** (`/`) renders a client-side shell with Resume / Work / Posts tabs and an in-shell Contact view.
- **Static site**: `next.config.ts` sets `output: 'export'`; deployable files land in `out/`. No backend, database, or required environment variables.
- Content is mostly typed, hard-coded data in `lib/content.ts`; the GitHub contribution heatmap is a generated snapshot in `lib/contributions.json`; public GitHub repos are fetched at runtime in the browser.

## 2. Stack and commands

| Area | Choice |
| --- | --- |
| Framework | Next.js 15 App Router (lock-resolved 15.5.9), React 18.3.1 |
| Language | TypeScript 5.8.3, strict config, `@/*` → repo root |
| Styling | Tailwind CSS 3.4.17, CSS-variable design tokens, `next-themes`, `tailwindcss-animate` |
| Motion | `framer-motion` 12.29.0 |
| UI primitives | shadcn/ui + Radix (`components/ui/`), `lucide-react`, `simple-icons` |
| Data | TanStack Query provider is mounted, but repo loading uses plain `fetch` |
| SEO | `metadata` in `app/layout.tsx`, JSON-LD, `app/robots.ts`, `app/sitemap.ts`, `next-sitemap` |
| Package manager | npm (`package-lock.json` is current; `bun.lockb` is stale) |

```bash
npm install
npm run dev        # http://localhost:3000
npm run build      # static export into out/; npm also runs postbuild -> next-sitemap
npm run lint       # next lint (deprecated in installed Next 15.5.9)
npx eslint .       # preferred direct flat-config lint
npx tsc --noEmit   # manual typecheck; build deliberately ignores TS errors
```

Observed local environment: Node v24.22.3 / npm 11.19.1. There is no `engines` or `packageManager` field.

## 3. Project structure

```text
.
├── app/
│   ├── globals.css        # Tailwind layers, light/dark tokens, base + utility CSS
│   ├── layout.tsx         # server root layout, metadata, fonts, providers, toasters
│   ├── page.tsx           # "use client" — the entire interactive shell
│   ├── providers.tsx      # QueryClientProvider
│   ├── robots.ts          # force-static /robots.txt
│   └── sitemap.ts         # force-static /sitemap.xml (single URL)
├── components/
│   ├── seo/StructuredData.tsx   # client-injected Person + ProfessionalService JSON-LD
│   ├── shell/                   # bespoke portfolio UI (17 components)
│   │   ├── Sidebar.tsx, MobileBar.tsx        # shared navItems + ThemeToggle
│   │   ├── ProfileHero.tsx, RightRail.tsx
│   │   ├── ResumeTab.tsx, WorkTab.tsx, PostsTab.tsx
│   │   ├── ContactPage.tsx, ContactBand.tsx
│   │   ├── ProductCard.tsx, ContributionBanner.tsx
│   │   └── Chip.tsx, ToolChip.tsx, Rich.tsx, Reveal.tsx, CountUp.tsx, LocalClock.tsx
│   ├── ui/                # 49 generated shadcn/ui files; only 4 used externally
│   └── theme-provider.tsx # unused wrapper (layout imports next-themes directly)
├── hooks/
│   ├── use-github-repos.ts # client fetch of public GitHub repos
│   ├── use-mobile.tsx      # md breakpoint matchMedia
│   └── use-toast.ts        # legacy shadcn toast store
├── lib/
│   ├── content.ts          # ALL portfolio copy/data; **text** = emphasis
│   ├── contributions.json  # generated GitHub contribution snapshot
│   ├── tool-icons.ts       # simple-icons + hand-made brand glyph fallbacks
│   └── utils.ts            # cn() = twMerge(clsx(...))
├── public/
│   ├── images/products/*   # dentora, larameet, lara-sequence, larasoft
│   ├── images/projects/*   # 11 project screenshots
│   ├── favicon.png, placeholder.svg
│   └── robots.txt, sitemap.xml  # tracked, stale SEO files (see gotchas)
├── scripts/fetch-contributions.mjs # `gh` GraphQL -> lib/contributions.json
├── components.json         # shadcn config (tailwind.css path is wrong)
├── next.config.ts, next-sitemap.config.js, tailwind.config.ts, postcss.config.mjs
├── tsconfig.json, eslint.config.js, global.d.ts
├── DEPLOYMENT.md           # static-hosting instructions
└── README.md               # STALE Lovable/Vite template — do not trust
```

Generated/disposable: `.next/`, `out/`, `next-env.d.ts`, `tsconfig.tsbuildinfo`. Never hand-edit them.

## 4. Architecture quick map

### Routing and navigation

- `app/layout.tsx` is the only server component route shell; `app/page.tsx` is `"use client"` and owns all state.
- Tabs are **not routes or query params**. `app/page.tsx` reads `location.hash` (`#profile`, `#resume`, `#work`, `#posts`, `#contact`) via `syncFromHash()` and a `hashchange` listener.
- `choose()` uses `history.replaceState`, so tab switches do not add back-button history.
- `#resume`, `#work`, `#posts`, and `#contact` are not real element IDs; the handler scrolls to `#tabs` itself.
- Sidebar collapsed state is persisted in `localStorage["sidebar"]`, with access wrapped in `try/catch`.
- Nav links come from `navItems` in `components/shell/Sidebar.tsx`, reused by `MobileBar.tsx`.

### Content and data flow

- `lib/content.ts` exports: `experiences`, `education`, `projects`, `products`, `about`, `skills`, `linkedinPosts`, `technologies`, `profile`, `stats`, `socialLinks`, `repoFallbackDescriptions`, `projectCategories`, `liveProjects`, `topSkills`, `aiToolsShort`, `stackShort`, `hiddenRepos`, `workWithMe`.
- `**double asterisks**` in text are rendered as emphasis by `components/shell/Rich.tsx`.
- `hooks/use-github-repos.ts` fetches up to 5 pages of unauthenticated GitHub API results, filters forks/archived/own-profile repo, sorts by stars then push date. `WorkTab.tsx` then applies `hiddenRepos`, shows 6 repos, and offers a "Show all" toggle. Failures show a GitHub link fallback; descriptions fall back to `repoFallbackDescriptions`.
- `lib/contributions.json` is a dated snapshot. Regenerate with `node scripts/fetch-contributions.mjs [login]` (needs signed-in `gh` CLI), then rebuild.

### Components, styling, animation

- `components/shell/*` is the real UI layer; `components/ui/*` is mostly unused shadcn scaffolding.
- Design tokens live in `app/globals.css` (`:root` and `.dark`); `tailwind.config.ts` maps them to Tailwind color names. Accent is orange; radius token is `--radius: 0.875rem`.
- Fonts are `next/font/google` DM Sans + JetBrains Mono, exposed as `--font-sans` and `--font-jetbrains-mono`.
- Theme: `next-themes` with `attribute="class"`, default light, system opt-in; only control is `ThemeToggle` in `Sidebar.tsx`.
- Reusable motion: `Reveal` (whileInView), `CountUp`, `MotionConfig reducedMotion="user"` in `page.tsx`, `AnimatePresence` for tab transitions and project filtering.
- Icons: `lucide-react` for UI/social; `simple-icons` + `ToolChip` fallbacks for tech logos. Raw `<img>` is used intentionally; images are unoptimized for static export.

### SEO and deployment

- `app/layout.tsx` holds primary metadata; `components/seo/StructuredData.tsx` injects JSON-LD client-side.
- `app/sitemap.ts` and `app/robots.ts` are `force-static` and write into `out/`.
- `postbuild` runs `next-sitemap`, whose defaults write `public/sitemap.xml` and `public/robots.txt`. Those tracked files are stale and may be overwritten by a build.
- Deploy the contents of `out/` to any static host. `DEPLOYMENT.md` covers cPanel/FTP, Netlify, Vercel, and GitHub Pages. GitHub Pages subpath hosting would require `basePath`/asset-path changes because URLs are absolute and `metadataBase` is hard-coded to `https://farookhridoy.com`.

## 5. Development skills — task playbooks

### Run and verify locally

1. `npm install` then `npm run dev`.
2. Before committing: `npx tsc --noEmit` and `npx eslint .` (build does **not** fail on TS/ESLint errors).
3. `npm run build`, confirm `out/index.html`, `out/_next/`, `out/robots.txt`, `out/sitemap.xml`.

### Change portfolio copy or profile facts

1. Edit `lib/content.ts` (`profile`, `about`, `experiences`, `education`, `skills`, `stats`, etc.).
2. Use `**emphasis**` only where `Rich.tsx` renders it.
3. Mirror name/title/description changes in `app/layout.tsx` metadata and `components/seo/StructuredData.tsx`.
4. Update `lib/tool-icons.ts` if adding a technology chip.

### Add a product, live project, or repo entry

- Product: add to `products` in `lib/content.ts` and put the image in `public/images/products/`.
- Live project: add to `liveProjects`, using a `ProjectCategory` from `projectCategories`; image goes in `public/images/projects/`.
- Legacy project card: add to `projects` (supports `description` and `tech`).
- Hide a GitHub repo: add its name to `hiddenRepos`.
- Fix a repo with no GitHub description: add a key to `repoFallbackDescriptions`.

### Change contact/social links

- Edit `socialLinks` and `workWithMe` in `lib/content.ts`.
- If adding a new social id, add a matching Lucide icon to `socialIcons` in `components/shell/RightRail.tsx`.
- `ContactPage.tsx` uses non-null assertions on `socialLinks.find(...)`; keep ids in sync or it can crash.
- Verify the WhatsApp handle `farookhriody` (possible typo) before publishing.

### Add a new tab or navigation section

1. Extend `Tab`/`tabs` in `app/page.tsx` and the render branch that selects the component.
2. Extend `SectionId` and `navItems` in `components/shell/Sidebar.tsx` (`MobileBar` reuses them).
3. Teach `syncFromHash()` about the new hash; remember these hashes are state, not element ids.
4. Scroll target is `#tabs`; add a real DOM id only if you also update the observer logic.

### Change theme or branding

- Colors/radius: `app/globals.css` → `:root` and `.dark`.
- Tailwind mapping/keyframes/plugins: `tailwind.config.ts`.
- Fonts: `app/layout.tsx` (`next/font/google` + CSS variables).
- Keep contrast in mind: `lib/tool-icons.ts` switches brand icons to `currentColor` on light backgrounds.

### Add or edit UI components

- Bespoke portfolio UI → `components/shell/` (PascalCase file, PascalCase component).
- Reusable primitives → consider shadcn CLI; config is `components.json`, but its `tailwind.css` path says `src/index.css` while the real file is `app/globals.css`. Fix or ignore consciously.
- Any component using hooks/context/browser APIs needs `"use client"` on line 1. Most generated `ui/*` files lack it and are only safe because they are unused in server components.
- Prefer the `@/` alias for imports.

### Refresh the contribution heatmap

```bash
gh auth login            # if needed
node scripts/fetch-contributions.mjs farookhridoy
npm run build
```

The script is not wired into `package.json` and requires the signed-in `gh` CLI with GraphQL access.

### Update SEO / sitemap

- Primary metadata: `app/layout.tsx`; JSON-LD: `components/seo/StructuredData.tsx`.
- Single route sitemap: `app/sitemap.ts`; allow-all robots: `app/robots.ts`.
- `next-sitemap.config.js` reads `SITE_URL` or falls back to `https://farookhridoy.com`.
- Decide intentionally whether tracked `public/robots.txt` / `public/sitemap.xml` should be refreshed by `postbuild`; `out/` versions come from app routes and currently differ.

### Deploy

1. `npm run build`.
2. Upload the **contents of `out/`** (not the folder itself) to the static host.
3. Test `/`, hash links, `/robots.txt`, `/sitemap.xml`, images, and theme toggle.
4. `npm run start` is not the normal serving path with `output: 'export'`; serve `out/` statically.

## 6. Conventions

- Shell components: PascalCase filenames. Hooks/libs: kebab-case. shadcn UI: lowercase.
- `"use client"` goes on line 1; server files have no directive. There is no `"use server"`.
- Path alias `@/*` maps to repo root; config files often use single quotes while source components mostly use double quotes.
- `app/page.tsx` is intentionally one large client shell; avoid splitting into routes unless you also redesign hash navigation.
- Accessibility already in place: skip link, `aria-current`, `aria-pressed`, `aria-expanded`, focus ring, `sr-only` headings, reduced-motion support. Preserve them.

## 7. Gotchas / maintenance backlog

- `README.md` is an untouched Lovable/Vite template; do not use it as project documentation.
- Two lockfiles exist: `package-lock.json` (current) and `bun.lockb` (old template). Use npm unless intentionally migrating.
- 49 shadcn/ui files exist but only `toaster`, `sonner`, `tooltip`, and `avatar` are imported outside `components/ui/`. Many dependencies exist only for unused components.
- Two toast systems are mounted (`Toaster` and `Sonner`) and no app code calls `toast()`. `components/ui/use-toast.ts` is a redundant re-export.
- `components/theme-provider.tsx` is unused; `global.d.ts` CSS module declarations are unused.
- `app/layout.tsx` declares `/favicon.ico`, but `public/` only has `favicon.png`.
- Tabs expose ARIA roles but lack arrow-key navigation/roving tabindex; `aria-controls` points to a panel that is not rendered until selected. `#contact:target` CSS is effectively dead because there is no `id="contact"` element.
- `useGithubRepos` bypasses the mounted `QueryClient`; no cache/dedupe/token, so API rate limits or network errors degrade the Open source section.
- `next.config.ts` disables ESLint and TypeScript errors during build. `npm run lint` uses deprecated `next lint`.
- No tests, no test script, no dedicated typecheck script. Treat `npx tsc --noEmit` + `npx eslint .` as the real checks.
- GitHub Pages deployment as described in `DEPLOYMENT.md` needs `basePath`/asset handling for a subpath.
