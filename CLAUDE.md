# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AkamonLabs brand website (akamonlabs.com) — a Hugo static site deployed via GitHub Pages. Serves as landing pages for iOS/Android/Mac apps (e.g., akamonlabs.com/stagewhisper) with associated legal pages (privacy policy, terms).

## Technology Stack

- **SSG**: Hugo (extended edition, v0.154+)
- **Theme**: PaperMod via Hugo Modules (not git submodules)
- **Language**: Go 1.21+ (for Hugo Modules)
- **Hosting**: GitHub Pages
- **CI/CD**: GitHub Actions (`.github/workflows/hugo.yaml`)

## Common Commands

```bash
# Local development server
hugo server -D

# Production build
hugo --gc --minify

# Initialize Hugo Modules (first time setup)
hugo mod init github.com/scarrilho/akamonlabs.com
hugo mod get github.com/adityatelange/hugo-PaperMod

# Update theme module
hugo mod get -u

# Create new content
hugo new content/<section>/index.md
```

## Architecture

### Content Structure
- All content uses Hugo **Page Bundles** (folder + `index.md` + co-located assets)
- App landing pages live at `content/<app-name>/index.md` (e.g., `content/stagewhisper/index.md`)
- Legal pages (privacy policy, terms) are nested under app sections
- Data-driven sections use co-located YAML files + custom layout templates

### Key Directories
- `content/` — site content (Markdown + data files)
- `layouts/` — custom template overrides (PaperMod provides `extend_head.html` and `extend_footer.html` hooks)
- `assets/css/extended/` — custom CSS (auto-loaded by PaperMod)
- `static/` — files copied as-is to output (CNAME, images, favicons)
- `static/CNAME` — custom domain file for GitHub Pages

### Hugo Configuration
- Main config: `hugo.toml`
- Theme referenced as Hugo Module: `theme = ["github.com/adityatelange/hugo-PaperMod"]`
- Module dependencies managed in `go.mod` / `go.sum`

### Data-Driven Template Pattern
For sections rendered from YAML data (features, app lists):
```go-template
{{ $data := .Resources.GetMatch "data.yaml" | transform.Unmarshal }}
{{ range $data }}
  ...
{{ end }}
```

## Reference
- `page-requirement.md` — project goals and requirements
- `page-reference-architecture.md` — detailed architecture blueprint from sergiocarrilho.com (the reference site this project is based on)
