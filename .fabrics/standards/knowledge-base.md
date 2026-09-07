# Project Knowledge Base — giveexamples.com

## 1. Architecture Overview

Personal technical blog built as a Hugo static site. No backend, no database. Content is Markdown files compiled to static HTML, deployed to GitHub Pages via GitHub Actions on every push to `main`.

Key integration points:
- **Giscus** — GitHub Discussions-backed comments, loaded lazily per post
- **Google Analytics** — GA4 tag (G-J3LM16S9PR) via custom partial
- **Fuse.js** — client-side full-text search over a generated `index.json`
- **PaperMod** — Hugo theme as a git submodule

## 2. Tech Stack and Versions

| Component | Version / Detail |
|-----------|-----------------|
| Hugo | 0.146.0 (extended, pinned in CI) |
| Theme | PaperMod (git submodule at `themes/PaperMod`) |
| Search | Fuse.js 6.6.2 (CDN) |
| Node.js | 20 (CI only, for Hugo extended pipeline) |
| Deployment | GitHub Actions → GitHub Pages |
| Domain | giveexamples.com (CNAME in `static/`) |

Not using: Hugo Modules (uses git submodule instead), any SSR/ISR, CMS, or build-time API calls.

## 3. Curated Knowledge Sources

- [Hugo docs](https://gohugo.io/documentation/) — templating, content organization, shortcodes
- [PaperMod wiki](https://github.com/adityatelange/hugo-PaperMod/wiki) — theme config, params, overrides
- [Giscus docs](https://giscus.app/) — comment widget config
- [Hugo PaperMod extend_head/extend_footer](https://github.com/adityatelange/hugo-PaperMod/wiki/FAQs#custom-head--footer) — how to inject custom CSS/JS

## 4. Project Structure

```
blog/
├── hugo.toml                  # Site config: baseURL, theme, menus, params, Giscus
├── content/
│   ├── _index.md              # Homepage content (Featured Series + Recent Articles sections)
│   ├── about.md               # About page (TOML frontmatter — inconsistency vs rest)
│   ├── articles.md            # Articles index page
│   ├── archives.md            # Archives page
│   ├── search.md              # Search page
│   ├── impressum.md           # Legal page
│   ├── identity-fundamentals/ # Series: auth/authz, OAuth2/OIDC/SAML, tokens, workload identity
│   ├── application-integration/ # Series: Spring Boot + Entra ID, Azure CLI + Terraform
│   ├── performance-optimization/ # Series: N+1 query problem
│   ├── artificial-intelligence/ # Series: MCP servers
│   ├── judo/                  # Series: Kuzushi article
│   └── ai/                    # Empty — registered in mainSections but has no content yet
├── layouts/
│   ├── _default/
│   │   └── list.html          # Custom list: homepage shows last 5 posts + "View All" link
│   ├── partials/
│   │   ├── extend_head.html   # Custom CSS (search modal, tags, series cards, back-link) + Schema.org JSON-LD
│   │   ├── extend_footer.html # Search modal HTML + Fuse.js search logic
│   │   ├── comments.html      # Giscus embed (reads params from hugo.toml)
│   │   └── google_analytics.html # GA4 script
│   └── identity-fundamentals/ # Section-specific layout overrides (list, terms, articles)
├── static/
│   ├── CNAME                  # giveexamples.com
│   ├── favicon.*              # ico, svg, 16x16, 32x32
│   └── images/                # Static images
├── assets/css/                # Custom CSS (processed by Hugo pipes)
├── .github/workflows/hugo.yml # CI: build + deploy to GitHub Pages
└── .gitmodules                # PaperMod submodule reference
```

## 5. Project Conventions

**Content frontmatter (YAML, `---`):**
- Required: `title`, `date`, `draft`, `description`
- Series articles use `weight` for ordering within a section
- Per-post feature flags: `comments: true`, `ShowReadingTime: true`, `ShowShareButtons: true`, `ShowPostNavLinks: true`
- Tags are lowercase kebab-case: `["oauth2", "spring-boot", "api-security"]`
- `about.md` uses TOML (`+++`) — exception, everything else uses YAML

**Adding a new content section:**
1. Create `content/<section-name>/` directory
2. Add `_index.md` with title + description (currently missing on most sections — impacts SEO)
3. Add section slug to `mainSections` in `hugo.toml` if it should appear on homepage
4. Add a menu entry in `hugo.toml` if it needs top-nav visibility

**Custom layouts override PaperMod** via Hugo's lookup order — files in `layouts/` shadow theme files without modifying the submodule.

**Search** is powered by `home = ["HTML", "RSS", "JSON"]` output format — the `index.json` is generated at build time and fetched client-side by Fuse.js on modal open.

**Comments** are opt-in per post via `comments: true` in frontmatter — the `comments.html` partial checks `site.Params.giscus` before rendering.

**CI deploys automatically** on push to `main` — no staging environment; `public/` is gitignored and built fresh each run.
