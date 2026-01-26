## Project Overview

This is a personal blog built with Hugo (static site generator) using the PaperMod theme. The site is published at https://www.aziogas.lt/ and content is in Lithuanian.

## Commands

```zsh
# Build static site to /public/
hugo

# Serve all the blog locally
hugo serve

# Include draft posts
hugo serve -D

# Check Hugo version
hugo version
```

## Architecture

```
content/
├── post/          # Blog articles (Markdown)
└── page/          # Static pages (about.md, contact.md)

config/_default/
└── config.yaml    # Main site configuration

layouts/
├── partials/      # Custom theme overrides (footer, breadcrumbs, newsletter)
└── shortcodes/    # Custom content elements (pdf, collapse, super)

static/            # Images, PDFs, videos served as-is

i18n/
└── lt.yaml        # Lithuanian UI translations

themes/PaperMod/   # Theme (external, do not modify directly)
```

## Key Configuration

- **Base URL**: https://www.aziogas.lt/
- **Default language**: Lithuanian (`lt`)
- **Permalinks**: `/:filename/` (clean URLs from filename)

## Content Workflow

New posts go in `content/post/` as Markdown files with Hugo frontmatter:

```yaml
---
title: "Post Title"
date: 2024-01-01
draft: false
tags: ["tag1", "tag2"]
---
```

## Custom Shortcodes

- `{{< pdf src="/file.pdf" >}}` - Embed PDF documents
- `{{< collapse title="Title" >}}content{{< /collapse >}}` - Collapsible sections
- `{{< super >}}text{{< /super >}}` - Superscript text

## Issues

If port 1313 is in use: `lsof -i :1313` to find the process, then `kill -9 <PID>`.

## Writing Style Guide

All content is written in Lithuanian. Follow these conventions:

### Tone & Voice

- **Friendly and conversational** — address the reader directly using informal "tu" form
- **Opening**: Often start with "Labas!" or dive straight into the topic
- **Closing phrases**: End with expressions like "Štai ir viskas!", "Voilà!", "Bon voyage!", or "_May the force be with you_"

### Structure

- Reasearch and plan before you act
- Be clear for every user and don't miss necessary step for technical articles
- Explain the theory but also show practical examples
- Be brief, practical and close to the reader
- Use `###` headers for main sections
- For technical posts, include a "Šaltiniai" (Sources) section at the end with links
- For German-related posts, include "Aktualus vokiškas žodynas" vocabulary section

### Literary Endings

Technical articles often end with philosophical or literary quotes (from books, films, interviews) separated by `---`. These add personality and are italicized.

### Tone Examples

- ✓ "Labas! Tai galima pasiekti susikonfigūruojant..."
- ✓ "Voilà! Dabar mums nebereikės sukti galvos..."
- ✓ "Labas, šiandien noriu pasidalinti..."
- ✗ "Šiame straipsnyje bus aptariama..." (too formal/academic)

### Technical Articles

- Provide step-by-step instructions with numbered sections or clear headers
- Include code blocks with language tags (`bash`, `python`, `sql`, `ini`, etc.)
- Explain technical terms in parentheses, e.g., "(angl. *type-hint*)" or "(*vok*. vollmachtgebende Person)"
- Add troubleshooting section "Jei nepavyko" for complex setups
- Cross-reference related articles using internal links

### Frontmatter

```yaml
---
title: Short descriptive Lithuanian title
description: One-line summary in Lithuanian
date: 2024-01-01T04:00:00Z
author: Ąžuolas Krušna
tags: ["Tag1", "Tag2"]
ShowCodeCopyButtons: true  # For posts with code
---
```

