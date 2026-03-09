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

- Base URL: https://www.aziogas.lt/
- Default language: Lithuanian (`lt`)
- Permalinks: `/:contentbasename/` (clean URLs from filename)

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

## Writing Style Guide

All content is written in Lithuanian. Follow these conventions:

### Tone & Voice

- Friendly and conversational — address the reader directly using informal "tu" form
- Opening: Often start with "Labas!" or dive straight into the topic
- Closing phrases: End with expressions like "Štai ir viskas!", "Voilà!", "Bon voyage!" etc.
- Start with personal motivation — explain WHY you needed this before the how-to. A real story: bad internet during COVID → local Vertica, tired and made mistakes → squash commits, dentist in Berlin → how to call Germany

### Titles

- How-to posts use imperative verbs: Susikurk, Suspausk, Nuspalvink, Pakeisk, Pasileisk, Surask, Užsidėk, Paskambink
- Other posts use descriptive nouns: "Docker sąvokos lietuviškai", "Draugiškos Python klasės", "Sprintas ar bintas?"

### Structure

- Research and plan before you act
- Be clear for every user and don't miss necessary steps for technical articles
- Explain the theory but also show practical examples
- Be brief, practical and close to the reader
- Use `###` headers for main sections
- For longer posts, give the quick answer first, then offer an early exit: "Jeigu tiek ir reikėjo, tuomet užverčiame šį puslapį ir einam pasimėgauti likusia diena!" — then dive deeper for curious readers
- Longer posts get an "Apibendrinimas" (summary) section near the end, before the literary quote
- For technical posts, include a "Šaltiniai" (Sources) section at the end — use bare links: `- Title URL`, not markdown `[Title](URL)`
- For German-related posts, include "Aktualus vokiškas žodynas" vocabulary section

### Literary Endings

End technical articles with a quote — from a book, film, interview, whatever fits. Separate it with `---` and italicize it.

- Favorite: Matilda Olkinaitė poems (used in 5+ posts)
- Also used: Douglas Adams "Keliautojo autostopu gidas po galaktiką", Dino Buzzati, Arūnas Žebriūnas

### Tone Examples

- ✓ "Labas! Tai galima pasiekti susikonfigūruojant..."
- ✓ "Voilà! Dabar mums nebereikės sukti galvos..."
- ✓ "Labas, šiandien noriu pasidalinti..."
- ✗ "Šiame straipsnyje bus aptariama..." (too formal/academic)

### Technical Articles

- Provide step-by-step instructions with numbered sections or clear headers
- Include code blocks with language tags (`bash`, `python`, `sql`, `ini`, etc.)
- Explain technical terms in parentheses, e.g., "(angl. _type-hint_)" or "(_vok_. vollmachtgebende Person)"
- Add troubleshooting section "Jei nepavyko" for complex setups
- Cross-reference related articles using internal links

### Frontmatter

```yaml
---
title: Short descriptive Lithuanian title
description: One-line summary in Lithuanian
date: 2026-01-01T04:00:00Z
author: Ąžuolas Krušna
tags: ["Tag1", "Tag2"]
ShowCodeCopyButtons: true  # For posts with code
---
```
