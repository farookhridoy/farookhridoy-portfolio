# Project Analysis — terminal-dev-portfolio

Generated from a static read of the repository at `/Users/farookhridoy/Projects/www/terminal-dev-portfolio`. No source files were changed to produce this report. For task-oriented guidance see [`AGENTS.md`](./AGENTS.md).

---

## 1. Executive summary

This is a **single-page personal portfolio** for MD. Omar Farook Hridoy, built with the **Next.js 15 App Router** and deployed as a **fully static site** (`output: 'export'`). There is exactly one user-facing route, `/`; its entire interactive shell lives in the client component `app/page.tsx`. Resume, Work, and Posts are tabs within that shell, and Contact is an alternate view inside the same shell — none are separate routes.

The site has no backend or database. Content is primarily **hard-coded TypeScript data** in `lib/content.ts`, with two dynamic-ish inputs:

1. `lib/contributions.json` — a manually regenerated GitHub contribution snapshot rendered as a heatmap.
2. `hooks/use-github-repos.ts` — a browser-side `fetch` of the public GitHub API, filtered by `hiddenRepos` in `WorkTab.tsx`.

Configuration deliberately relaxes build gates (`eslint.ignoreDuringBuilds`, `typescript.ignoreBuildErrors`) and images are unoptimized for static hosting. The repository also contains substantial leftover template scaffolding (stale README, unused shadcn components, dual lockfiles, stale SEO files).

---

## 2. Technology stack

### Declared and resolved versions

`package.json` declares Next `^15.1.4`, React `^18.3.1`, TypeScript `^5.8.3`, Tailwind `^3.4.17`, framer-motion `^12.29.0`, next-themes `^0.3.0`, lucide-react `^0.462.0`, simple-icons `16.35.0` (pinned), TanStack Query `^5.83.0`, and next-sitemap `^4.2.3`. The npm lockfile resolves Next to **15.5.9**, React **18.3.1**, TypeScript **5.8.3**, Tailwind **3.4.17**, ESLint **9.32.0**.

Observed environment: Node **v24.22.3**, npm **11.19.1**. There is no `engines` or `packageManager` field.

### Scripts (`package.json:5-11`)

| Script | Command | Notes |
| --- | --- | --- |
| `dev` | `next dev` | Standard dev server |
| `build` | `next build` | npm automatically runs `postbuild` afterward |
| `start` | `next start` | Not the normal path with `output: 'export'`; serve `out/` statically |
| `lint` | `next lint` | Deprecated in installed Next 15.5.9 |
| `postbuild` | `next-sitemap` | Uses `next-sitemap.config.js`; may write tracked files under `public/` |

There is **no test script**, no `typecheck` script, and no test files.

### Configuration highlights (`next.config.ts`)

```ts
output: 'export',
reactStrictMode: true,
images: { domains: ['farookhridoy.com'], unoptimized: true },
eslint: { ignoreDuringBuilds: true },
typescript: { ignoreBuildErrors: true },
```

`images.domains` is legacy and inert because no `next/image` usage exists. Raw `<img>` elements are used, with lazy loading.

---

## 3. Source structure

```text
app/
  globals.css        Tailwind layers, light/dark tokens, base styles, utilities
  layout.tsx         Server root layout: metadata, fonts, providers, toasters
  page.tsx           "use client" — full interactive shell
  providers.tsx      QueryClientProvider
  robots.ts          force-static /robots.txt
  sitemap.ts         force-static /sitemap.xml

components/
  seo/StructuredData.tsx   Client-emitted Person + ProfessionalService JSON-LD
  shell/                   Bespoke portfolio components (17 files)
    Sidebar.tsx, MobileBar.tsx, ProfileHero.tsx, RightRail.tsx,
    ResumeTab.tsx, WorkTab.tsx, PostsTab.tsx,
    ContactPage.tsx, ContactBand.tsx, ProductCard.tsx,
    ContributionBanner.tsx, Chip.tsx, ToolChip.tsx, Rich.tsx,
    Reveal.tsx, CountUp.tsx, LocalClock.tsx
  ui/                      49 shadcn/ui files; only 4 used externally
  theme-provider.tsx       Unused next-themes wrapper

hooks/
  use-github-repos.ts      Browser fetch of GitHub repos
  use-mobile.tsx           md breakpoint via matchMedia
  use-toast.ts             Legacy shadcn toast reducer/store

lib/
  content.ts               All portfolio copy/data
  contributions.json       Generated contribution snapshot
  tool-icons.ts            Brand-glyph mapping
  utils.ts                 cn()

public/
  images/products/         4 product images
  images/projects/         11 project screenshots
  favicon.png, placeholder.svg
  robots.txt, sitemap.xml  Tracked, stale SEO files

scripts/fetch-contributions.mjs  gh GraphQL -> lib/contributions.json

Root configs: components.json, eslint.config.js, global.d.ts,
next-sitemap.config.js, next.config.ts, package.json,
package-lock.json, bun.lockb, postcss.config.mjs,
tailwind.config.ts, tsconfig.json, README.md, DEPLOYMENT.md
```

Generated and disposable: `.next/`, `out/`, `next-env.d.ts`, `tsconfig.tsbuildinfo`.

---

## 4. Routing, server/client boundary, and state

### Routes

Only three App Router files produce routes:

- `app/page.tsx` → `/`, the whole application.
- `app/robots.ts` → `/robots.txt`, `force-static`, allow all, sitemap at `https://farookhridoy.com/sitemap.xml`.
- `app/sitemap.ts` → `/sitemap.xml`, `force-static`, one absolute URL `/`, `changeFrequency: 'monthly'`, `priority: 1`, `lastModified: new Date()` (changes every build).

There are no nested layouts, loading/error boundaries, or additional pages. `next/navigation` is not used.

### Root layout (`app/layout.tsx`)

- Server component.
- `metadata`: `metadataBase`, title default/template, description, 14 keywords, authors/creator, icons (`/favicon.png`, `/favicon.ico`), OpenGraph, Twitter `summary_large_image`, robots/googleBot directives.
- Fonts via `next/font/google`: `DM_Sans` → `--font-sans`, `JetBrains_Mono` → `--font-jetbrains-mono`, applied to `<body>`.
- Provider stack: `<html lang="en" suppressHydrationWarning>` → `ThemeProvider` from `next-themes` → `Providers` (QueryClient) → `TooltipProvider` → children + `<Toaster />` + `<Sonner />`.
- The local `components/theme-provider.tsx` wrapper is not imported anywhere.

### Client boundary

`"use client"` appears at line 1 of:

- `app/page.tsx`, `app/providers.tsx`
- `components/seo/StructuredData.tsx`
- Shell: `ContactBand`, `ContactPage`, `CountUp`, `LocalClock`, `MobileBar`, `ProfileHero`, `Reveal`, `Sidebar`, `WorkTab`
- `components/theme-provider.tsx`, `components/ui/sonner.tsx`, `components/ui/toaster.tsx`
- `hooks/use-github-repos.ts`, `hooks/use-mobile.tsx`, `hooks/use-toast.ts`

Server components: `app/layout.tsx`, `app/robots.ts`, `app/sitemap.ts`, and shell `ResumeTab`, `PostsTab`, `RightRail`, `ContributionBanner`, `ProductCard`, `Chip`, `Rich`, `ToolChip`. They become part of the client graph when imported by client components (e.g. `ProfileHero` renders `ContributionBanner`; `WorkTab` renders `ProductCard`).

`StructuredData` is a client component using `dangerouslySetInnerHTML`, so JSON-LD is injected client-side rather than emitted as server metadata.

### Tab/navigation state (`app/page.tsx`)

- `type Tab = "resume" | "work" | "posts"`, `tabs` array, `isTab()` guard (lines 16-23).
- State: `tab`, `active: SectionId`, `view: "profile" | "contact"`, `collapsed` (lines 26-30).
- Sidebar collapse persisted in `localStorage["sidebar"]` with guarded reads/writes (32-48).
- `syncFromHash()` (51-69) interprets the URL fragment:
  - `#contact` → contact view, scroll to top.
  - `#resume` / `#work` / `#posts` → profile view, set tab, then `requestAnimationFrame(() => document.getElementById("tabs")?.scrollIntoView())`. The code comment explicitly notes these hashes are **not element IDs**.
  - `#profile` → active profile, scroll top.
- `hashchange` listener added/removed in an effect (71-75).
- `IntersectionObserver` on `#profile` and `#tabs` with `rootMargin: "-40% 0px -55% 0px"` keeps the highlighted nav item synced with scrolling (77-94).
- `choose()` sets the tab, calls `history.replaceState(null, "", "#" + next)`, sets active state, and scrolls to `#tabs` when needed (96-103). Tab clicks therefore do not create back-button history entries.
- Layout grid: sidebar `16rem` → `4.5rem` when collapsed; third `21rem` right rail appears at `xl`. Mobile bar and bottom nav are `lg:hidden`; right rail is sticky at `xl`.

Accessibility already present: skip link, `role="tablist"/"tab"/"tabpanel"`, `aria-selected`, `aria-controls`, `aria-labelledby`, `aria-current`, `aria-pressed`, `aria-expanded`, global focus ring, `sr-only` contact heading, `MotionConfig reducedMotion="user"`.

Known gaps: no arrow-key/roving-tabindex tab behavior; `aria-controls` references a panel that is not rendered until selected; no element has `id="contact"`, making `#contact:target > div` CSS in `globals.css:119-125` unreachable.

---

## 5. Content and data model

### `lib/content.ts` (415 lines)

Header comment: all portfolio content lives here; `**double asterisks**` mark emphasis rendered by `components/shell/Rich.tsx`.

| Lines | Export | Shape / notes |
| --- | --- | --- |
| 5-59 | `experiences` | 4 entries: `title`, `company`, `period`, `mark`, `description`, `activities[]`, `highlights[]` |
| 61-76 | `education` | 2 entries: `degree`, `institution`, `period`, `cgpa`, `type` |
| 78-104 | `projects` | 19 entries: `title`, `line`, optional `description`, `tech[]`, `type` |
| 107-112 | `products` | `as const`; 4 entries: `name`, `line`, `tags`, `image`, `href` |
| 114-119 | `about` | 4 paragraphs |
| 121-205 | `skills` | Grouped `{name, level}[]`: backend, frontend, database, server, devOps, tools, methodologies, softSkills, languages, aiTools |
| 207-211 | `linkedinPosts` | 3 `{title, id}` mapped to LinkedIn share URLs |
| 213-230 | `technologies` | 16 `{name, category}` |
| 232-246 | `profile` | Name, founder/co-founder info, title, tagline, summary, specialization, location, `timeZone: "Asia/Dhaka"`, remote GitHub avatar, site, `githubUser`, current role/company |
| 248-253 | `stats` | 4 `{value, label}` |
| 255-263 | `socialLinks` | `as const`; email, WhatsApp, GitHub, LinkedIn, X, Facebook, Instagram |
| 266-270 | `repoFallbackDescriptions` | Descriptions for repos without GitHub descriptions |
| 273-274 | `projectCategories` / `ProjectCategory` | `["E-commerce","EdTech","Corporate & real estate","SaaS & tools"]` |
| 276-372 | `liveProjects` | 11 typed `{name, category, line, tags[], image, href}` |
| 375-377 | `topSkills`, `aiToolsShort`, `stackShort` | Short display lists |
| 380-389 | `hiddenRepos` | 8 repos hidden from the Open source list |
| 392-415 | `workWithMe` | `as const`; main card + action cards |

There is no MDX or content-schema validation. Zod is installed but only referenced by unused shadcn `form.tsx`.

Potential typo: the WhatsApp entry uses `farookhriody` while most identifiers use `farookhridoy` (`lib/content.ts:257`).

### Contribution snapshot

`lib/contributions.json` is a single minified line:

```json
{"login":"farookhridoy","total":2575,"from":"2025-10-12","to":"2026-10-16","weeks":[[level,count],...]}
```

`components/shell/ContributionBanner.tsx` casts it to a calendar type and renders a CSS grid of cells (`grid-flow-col`, 7 rows) with level colors, staggered animation, tooltips, and `role="img"` + `aria-label`.

`scripts/fetch-contributions.mjs` is not wired into npm scripts. It requires a signed-in `gh` CLI, calls GitHub GraphQL, maps `NONE…FOURTH_QUARTILE` to 0-4, and overwrites `lib/contributions.json`. Regenerate manually:

```bash
node scripts/fetch-contributions.mjs [login]
npm run build
```

### GitHub repository fetching

`hooks/use-github-repos.ts` (`"use client"`):

- `Repo` type: `name`, `description: string | null`, `html_url`, `language: string | null`, `stargazers_count`, `pushed_at`.
- `State`: `{status:"loading"}` / `{status:"error"}` / `{status:"ready", repos: Repo[]}`.
- Plain `useEffect` + `fetch`, **not** React Query, despite `QueryClientProvider` in `app/providers.tsx`.
- Up to 5 pages, `per_page=100`, `sort=pushed`; stops on a short page. `AbortController` cancels on unmount/change.
- Filters forks, archived repos, and the profile README repo; sorts by stars desc then `pushed_at` desc.
- Errors set `status:"error"` except `AbortError`.
- No offline/fallback repo list — only a GitHub link in `WorkTab.tsx:42-50` and per-repo description fallbacks.
- Unauthenticated API: rate limits can degrade this section.

`WorkTab.tsx` calls the hook with `profile.githubUser`, removes `hiddenRepos` via `useMemo`, shows 6 initially (`REPOS_FIRST = 6`), and has a "Show all" toggle. Repo metadata shows language and stars.

### Environment variables

The only `process.env` reference in source is `next-sitemap.config.js:3`:

```js
siteUrl: process.env.SITE_URL || 'https://farookhridoy.com'
```

No `.env` files are tracked; GitHub fetching needs no configured env var.

---

## 6. Component architecture

### Bespoke shell (`components/shell/`)

- `Sidebar.tsx`: client; exports `SectionId` and shared `navItems`; contains `ThemeToggle` using `useTheme` with a mounted guard to avoid hydration mismatch; desktop sticky sidebar, collapse toggle with `aria-expanded`, nav with `aria-current`.
- `MobileBar.tsx`: client; reuses `navItems`/`ThemeToggle`; sticky top header and fixed 5-column bottom nav, `lg:hidden`, safe-area padding.
- `ProfileHero.tsx`: client; local `useLocalTime` using `Intl.DateTimeFormat` with `profile.timeZone`, 30s interval; Web Share API with clipboard fallback; renders `ContributionBanner`, avatar, founder/co-founder chips, current role, tool chips, `CountUp` stats.
- `WorkTab.tsx`: client; four sections — products (`ProductCard`), category-filtered live projects with `AnimatePresence mode="popLayout"` and `motion.ul layout`, open source via `useGithubRepos`, and static projects with a featured first item.
- `ResumeTab.tsx`: server; About, Experience timeline, Education, Top skills, Skills and tools; uses `Reveal`, `Rich`, `Chip`, `ToolChip`, `LocalClock`.
- `PostsTab.tsx`: server; three LinkedIn links from `linkedinPosts`.
- `RightRail.tsx`: server; exports `socialIcons: Record<socialLinks id, LucideIcon>`; X is a literal letter because Lucide has no X logo; `compact` hides the "Say hello" card.
- `ContactPage.tsx`: client; hero card, `workWithMe` cards, all social links; uses `socialIcons` from `RightRail` and non-null assertions on `socialLinks.find(...)`.
- Presentational: `ContactBand`, `ProductCard`, `Chip`, `ToolChip`, `Rich`, `Reveal`, `CountUp`, `LocalClock`, `ContributionBanner`.

`ProductCard.tsx` uses a raw `<img>` with an eslint-disable comment and lazy loading. No `next/image` usage exists.

### shadcn/ui (`components/ui/`)

`components.json`: style `default`, `rsc: false`, TSX true, aliases `@/components`, `@/lib/utils`, `@/components/ui`, `@/lib`, `@/hooks`, baseColor `slate`, CSS variables true. Its `tailwind.css` field points to `src/index.css`, which does not exist; the real stylesheet is `app/globals.css`.

Only four UI modules are imported outside `components/ui`:

- `app/layout.tsx`: `toaster`, `sonner`, `tooltip`
- `components/shell/ProfileHero.tsx`: `avatar`

Everything else is generated-but-unused (accordion, dialog, forms, tables, charts, carousel, sidebar, etc.). `components/ui/use-toast.ts` is a 3-line re-export; the full 188-line reducer/store lives in `hooks/use-toast.ts` (`TOAST_LIMIT = 1`, `TOAST_REMOVE_DELAY = 1000000`). No application code calls `toast()`, yet both `<Toaster />` and `<Sonner />` are mounted.

Most `components/ui/*` files lack `"use client"` despite using hooks/context; they are safe only because they are not imported into server components.

---

## 7. Styling, theming, and animation

`app/globals.css` (143 lines):

- Tailwind base/components/utilities; `:root` light tokens and `.dark` overrides (HSL triplets for background, foreground, card, popover, primary, secondary, muted, accent, destructive, border, input, ring, success; `--radius: 0.875rem`).
- Accent is orange: `--primary: 14 80% 45%` light, `16 88% 62%` dark.
- Base layer: global `border-border`, smooth scrolling, body font, `.font-mono`, focus-visible ring, custom webkit scrollbar, reduced-motion override.
- Utilities: `#contact:target > div` flash (currently unreachable), `.banner-cell` staggered `cell-in`, `.eyebrow`.

`tailwind.config.ts`:

- `darkMode: ["class"]`.
- Content globs include a nonexistent `pages/` directory plus `components/` and `app/`.
- CSS-variable color mapping and radius from `--radius`.
- Accordion keyframes/animations.
- Plugins: `tailwindcss-animate`, `@tailwindcss/typography` (no `prose` classes used).

Theme: `next-themes` class attribute, light default, system opt-in; `ThemeToggle` in `Sidebar.tsx` is the only control.

Animation:

- `MotionConfig reducedMotion="user"` wraps the page.
- `Reveal`: `motion[as]`, initial opacity 0 / y 18, whileInView, `viewport.once`, 0.5s ease `[0.22,1,0.36,1]`, optional delay.
- Tab transitions: `AnimatePresence mode="wait"`, 0.18s fade/slide; contact view 0.25s.
- Live-project filtering: `AnimatePresence mode="popLayout"` + `layout`.
- `CountUp`: framer-motion `animate()` + `useInView` + `useReducedMotion`, preserves suffixes like `+`.
- CSS-only: contribution cell stagger, contact flash.

Icons:

- `lucide-react` for UI/navigation/social.
- `simple-icons` in `lib/tool-icons.ts` for Claude, Cursor, Docker, Git, GitHub Actions, Laravel, Next.js, Node.js, Nuxt, React, Tailwind, TypeScript, Vue; luminance check falls back to `currentColor` on dark backgrounds.
- `genericGlyph` + `ToolChip.tsx` provide hand-made fallbacks for ChatGPT, GitHub Copilot, AWS, PHP, JavaScript, MySQL.

---

## 8. SEO, build output, and deployment

`npm run build` runs `next build` then npm triggers `postbuild: next-sitemap`.

Two SEO systems coexist:

1. **App routes**: `app/sitemap.ts` and `app/robots.ts`, both `force-static`, writing into `out/`. `out/sitemap.xml` observed with a single `/` URL and build-time `lastmod`.
2. **next-sitemap**: `next-sitemap.config.js` uses `SITE_URL` or fallback `https://farookhridoy.com`, `generateRobotsTxt: true`, `generateIndexSitemap: false`, default `changefreq: 'daily'`, `priority: 0.7`. Its installed defaults use `outDir: 'public'`, so it can overwrite tracked `public/sitemap.xml` and `public/robots.txt`; those tracked files are stale relative to the app-route output (three URLs, `lastmod 2026-08-08`, `Host:` line in robots).

`DEPLOYMENT.md` describes static hosting of `out/` (index.html, 404.html, robots.txt, sitemap.xml, favicon.png, placeholder.svg, `_next/`), with cPanel/FTP, Netlify drag-and-drop, Vercel import, and GitHub Pages options. No Node/npm is needed on the host.

Caveats:

- `npm run start` is not the normal path with `output: 'export'`.
- GitHub Pages at `username.github.io/repo-name` needs `basePath` and asset-path handling; absolute `/images/...` paths and `metadataBase` assume a root domain.
- `app/layout.tsx` declares `/favicon.ico` but `public/` only has `favicon.png`.
- `DEPLOYMENT.md` does not mention `out/index.txt` (RSC payload) or `SITE_URL`.

---

## 9. Conventions

- Shell components: PascalCase files/components. Hooks/libs: kebab-case. shadcn UI: lowercase.
- Imports use `@/*` alias to repo root; a few relative imports exist (`app/providers.tsx`, shell intra-folder imports).
- `"use client"` on line 1 where present; no `"use server"`.
- Source components mostly use double quotes/semicolons; config files mostly single quotes — style is mixed.
- `app/page.tsx` is intentionally one large client shell; hash state drives tabs rather than routes.
- Preserve existing accessibility: skip link, ARIA labels, focus ring, reduced-motion handling.

---

## 10. Risks and maintenance backlog

1. **Documentation**: `README.md` is the untouched Lovable/Vite template with placeholder project IDs and Vite instructions. Do not trust it.
2. **Lockfiles**: `package-lock.json` is current; `bun.lockb` is from the template era and `bun` is not on PATH in the observed environment.
3. **Dead scaffolding**: 49 shadcn/ui files but only 4 imported externally; many dependencies exist solely for unused components. Two toast systems mounted; redundant `use-toast.ts`; unused `theme-provider.tsx`; unused `global.d.ts` CSS declarations.
4. **Build gates**: `eslint.ignoreDuringBuilds` and `typescript.ignoreBuildErrors` are true; `npm run lint` uses deprecated `next lint`. Use `npx tsc --noEmit` and `npx eslint .` as real checks.
5. **No tests**: no test runner, no test files.
6. **SEO duplication**: metadata/JSON-LD content must be kept in sync with `lib/content.ts`; dual sitemap/robots generation can produce conflicting tracked files.
7. **Contribution data freshness**: `lib/contributions.json` is a snapshot; it goes stale until `scripts/fetch-contributions.mjs` is rerun and the site rebuilt.
8. **GitHub API fragility**: unauthenticated runtime fetch with no cache; React Query provider is unused for this data path.
9. **ARIA gaps**: tabs lack arrow-key navigation/roving tabindex; `aria-controls` targets a panel that is only rendered after selection; dead `#contact:target` CSS.
10. **Potential typo**: WhatsApp handle `farookhriody` in `lib/content.ts:257` likely should be `farookhridoy`.
11. **Subpath hosting**: absolute asset paths and hard-coded domain break the documented GitHub Pages subpath scenario.
12. **Generated files**: `.next/`, `out/`, `next-env.d.ts`, `tsconfig.tsbuildinfo` are disposable; never hand-edit them.
