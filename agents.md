# Blog Agent Instructions

This file is the source of truth for AI assistants helping to write, edit, and publish content on [mbignell.com](https://mbignell.com). Update it as preferences evolve.

---

## Site Overview

- **Owner:** Mike Bignell — software engineer at Accenture
- **Platform:** Hugo with PaperMod theme, deployed to GitHub Pages
- **Domain:** mbignell.com (also mikezs.github.io)
- **Repository:** https://github.com/mikezs/mikezs.github.io (public)

The blog is a personal technical outlet. Posts cover software engineering, tooling, technology opinions, and occasional non-technical observations. The bar for publishing is "interesting enough that a future me or a peer would find it useful" — not academic rigour.

---

## How to Create a Post

All posts live in `content/posts/`. Each post is a single Markdown file.

### File naming

```
content/posts/YYYY-MM-DD-slug-in-kebab-case.md
```

Example: `content/posts/2026-04-20-switching-to-hugo.md`

### Front matter template

Every post must start with this TOML front matter block:

```markdown
---
title: "Human-Readable Title Here"
date: YYYY-MM-DDT00:00:00+00:00
draft: false
description: "One or two sentences summarising the post. Used in previews and meta tags."
tags: ["tag-one", "tag-two"]
categories: ["Category"]
showToc: true
---
```

**Rules:**
- `draft: true` to stage without publishing; `draft: false` to publish
- `date` must be ISO 8601 format — use the real date the post is written, not a future date
- `description` is used for SEO and the post card on the listing page — make it specific and useful
- Tags are lowercase kebab-case, singular where possible (e.g. `"go"` not `"golang"`, `"swift"` not `"Swift"`)
- Categories are title-case, broad groupings (e.g. `"Engineering"`, `"Tools"`, `"Opinion"`)

### Content structure

- Start the post body directly after the front matter — no need to repeat the title as an H1 (PaperMod renders it from front matter)
- Use `##` for top-level sections, `###` for subsections
- Keep paragraphs short — 3–5 sentences max
- Use fenced code blocks with language identifiers: ` ```go `, ` ```bash `, ` ```toml `, etc.
- Prefer concrete examples over abstract explanations

---

## Voice and Tone

Write in first person, informally but precisely. The target reader is a senior engineer — assume competence, skip basics, get to the point.

**Do:**
- Have an opinion and state it directly
- Use "I" — this is a personal blog
- Include real-world context (what problem prompted this, what was tried first)
- Link to relevant external resources sparingly (only when they add genuine value)
- End with a brief "so what" — a takeaway, a next step, or an open question

**Don't:**
- Use marketing language or hype ("game-changing", "revolutionise", "exciting")
- Add filler intro paragraphs ("In this post we will explore...")
- Pad word count — if a thought is done, stop writing
- Over-hedge ("this might not work for everyone") — just make the caveats specific if they matter

---

## Tags and Categories Reference

Seed list — extend as needed, but prefer reusing existing tags over creating new ones.

| Tag | Use for |
|---|---|
| `swift` | Apple platform development |
| `ios` | iOS-specific content |
| `go` | Go language |
| `python` | Python |
| `devtools` | Developer tooling, CLIs, editors |
| `ai` | AI/ML tools and models |
| `claude` | Claude / Anthropic specifically |
| `github` | GitHub, Actions, Pages |
| `hugo` | This site's own setup |
| `opinion` | Takes and hot takes |
| `career` | Work, industry, engineering culture |

| Category | Use for |
|---|---|
| `Engineering` | Technical posts about code or systems |
| `Tools` | Software tools, workflows, setup |
| `Opinion` | Commentary and takes |
| `Meta` | Posts about the blog itself |

---

## Images and Assets

- Store images in `static/images/YYYY-MM/` to keep them organised by month
- Reference in Markdown as `![alt text](/images/YYYY-MM/filename.png)`
- Optimise images before committing — aim for < 200 KB for a full-width image
- PaperMod supports a `cover` image in front matter:

```markdown
cover:
  image: "/images/2026-04/my-cover.png"
  alt: "Description of the image"
  relative: false
```

---

## Publishing Workflow

1. Create the post file in `content/posts/`
2. Set `draft: false` when ready to publish (or leave `draft: true` to preview locally first)
3. Commit and push to `main` — GitHub Actions builds and deploys automatically
4. The live site updates within ~2 minutes of the push

To preview locally before pushing:
```bash
hugo server --buildDrafts
```
Then visit http://localhost:1313.

---

## What to Update in This File

As the blog evolves, keep this file current. Good things to add:

- **Recurring series:** if a series of posts is started, document the naming convention and front matter for it
- **Tone corrections:** if a published post needed significant rewriting, note what was wrong
- **New tags/categories:** add them to the reference table as they're created
- **Structural preferences:** e.g. "always include a TL;DR section for posts over 800 words"
- **Off-limits topics:** anything that shouldn't appear on the blog

---

## Technical Notes for Agents

- The theme is PaperMod, installed as a git submodule at `themes/PaperMod/`. Do not modify files inside the theme directory — override via layouts/ or params in hugo.toml instead.
- The `.well-known/apple-app-site-association` file in `static/` must be preserved — it supports an associated iOS app.
- `static/CNAME` contains `mbignell.com` and must not be removed or modified.
- Hugo version in use: **0.160.1 extended**. The GitHub Actions workflow pins this version — update both places if upgrading.
- `buildDrafts = false` in hugo.toml means drafts never appear in production, even if accidentally pushed.
