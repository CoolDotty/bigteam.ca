# AGENTS.md — bigteam.ca

Repository for [bigteam.ca](https://bigteam.ca), a static site hosted on GitHub Pages.
Built with [Astro](https://astro.build).

---

## Project Type & Architecture

- **Astro static site** — zero client-side JavaScript by default, compiled to static HTML at build time.
- `src/pages/` contains `.astro` page components. Astro routes based on file path.
- `public/` contains static assets (e.g. `CNAME`) that are copied verbatim to `dist/` during build.
- `astro.config.mjs` configures the build. Currently set to `output: 'static'` (the default).
- The site is deployed to GitHub Pages from the `gh-pages` branch via a custom GitHub Actions workflow.

---

## Essential Commands

| Command | Purpose |
|---------|---------|
| `pnpm dev` | Start Astro dev server with HMR |
| `pnpm build` | Build static site to `dist/` (Astro static build) |
| `pnpm preview` | Preview the built `dist/` site locally |
| `pnpm install --frozen-lockfile` | Install dependencies (CI uses this) |

> **Package manager:** `pnpm@10.4.1` (pinned in `package.json` via `packageManager`).

---

## Build & Deploy

### Build
```bash
pnpm build
```
Runs `astro build`, outputting to `dist/`.

### Deploy
Deployment is fully automated via GitHub Actions (`.github/workflows/deploy.yml`):

- **Trigger:** push to `main` or manual (`workflow_dispatch`).
- **Runner:** `ubuntu-latest`
- **Permissions:** `contents: write` (required to push the `gh-pages` branch).
- **Concurrency group:** `pages` with `cancel-in-progress: true`.

The workflow runs a custom shell script that:
1. Checks out the repo.
2. Installs pnpm via corepack.
3. Runs `pnpm install --frozen-lockfile && pnpm build`.
4. Saves `dist/*` to `/tmp/deploy`.
5. Switches to the `gh-pages` branch (creating an orphan branch if it doesn't exist).
6. Wipes the branch, copies artifacts in, commits with a timestamp message, and **force-pushes** to `origin/gh-pages`.

> **Important:** The `gh-pages` branch is entirely managed by CI. Do not commit to it manually.

GitHub Pages is configured to serve from the **`gh-pages` branch** (not `main`).

---

## Code Organization

```
├── astro.config.mjs          # Astro configuration
├── package.json              # Project metadata & scripts
├── pnpm-lock.yaml            # Lockfile (CI uses --frozen-lockfile)
├── public/
│   └── CNAME                 # GitHub Pages custom domain config
├── src/
│   └── pages/
│       └── index.astro       # Homepage (Astro page component)
├── .github/
│   └── workflows/
│       └── deploy.yml        # CI deploy workflow
└── dist/                     # Build output (gitignored)
```

---

## Key Gotchas

- **CNAME must stay in `public/`**. Astro copies `public/` verbatim to `dist/`. If you move `CNAME` to repo root or `src/`, GitHub Pages will drop the custom domain mapping on next deploy.
- **Force push on deploy.** The CI force-pushes to `gh-pages`. This is intentional — the branch is disposable.
- **Lockfile exists now.** Since Astro is a dependency, `pnpm-lock.yaml` is generated and committed. CI runs `pnpm install --frozen-lockfile`; any dependency changes must update the lockfile.
- **No test, lint, or format commands** are configured.
- **Package manager divergence.** `packageManager` in `package.json` is pinned to `pnpm@10.4.1`, but CI runs `corepack prepare pnpm@latest --activate`, which may install a newer version.

---

## Editing Guidelines

- Pages go in `src/pages/`. File paths become URL paths (`src/pages/index.astro` → `/`).
- Static assets (images, `CNAME`, `robots.txt`, favicons) go in `public/`.
- Astro components use `.astro` syntax: frontmatter JS between `---` fences, HTML-like template below.
- If you add dependencies, run `pnpm install` locally to regenerate `pnpm-lock.yaml` so CI's `--frozen-lockfile` flag doesn't fail.
