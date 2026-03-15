## Project Overview

This is a personal blog built with Hugo (static site generator) using the PaperMod theme. The site is published at https://www.aziogas.lt/ and content is in Lithuanian as well as has English translations.

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

```bash
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
- Default language: Lithuanian (`lt`), English language is found under `/en/`
- Permalinks: `/:contentbasename/` (clean URLs from filename) for both Lithuanian, `/:slug/` for English posts to support the SEO

## Content Workflow

### Creating New Posts

Use the Hugo CLI to create new posts — it automatically applies the archetype template:

```zsh
# Create a new post
# Path is relative to content/ directory
hugo new post/your-post-title.md

# For English posts, add language suffix:
hugo new post/your-post-title.en.md
```

The archetype template at `archetypes/default.md` creates frontmatter with `title` from the filename, `date`, `author`, empty list of `tags`, and `draft: true`. 

Adding images: Use relative paths with `..` to go up one level from the post to reach them. Posts, as well as images are found directly on the domain, e.g. https://www.aziogas.lt/uzsidek-domena-savo-blogui-ant-netlify/ and https://www.aziogas.lt/netlify-domain.png.

```markdown
![alt](../image.jpg)
```

## Custom Shortcodes

- `{{< pdf src="/file.pdf" >}}` - Embed PDF documents
- `{{< collapse title="Title" >}}content{{< /collapse >}}` - Collapsible sections
- `{{< super >}}text{{< /super >}}` - Superscript text

## Writing Style Guide

Below we can find recommendations

### Tone & Voice

- Friendly and conversational — address the reader directly using informal "tu" form
- Opening: Often start with "Labas!" or dive straight into the topic
- Closing phrases: End with expressions like "Štai ir viskas!", "Voilà!", "Bon voyage!" etc.
- Start with personal motivation — explain WHY you needed this before the how-to. A real story: bad internet during COVID → local Vertica, tired and made mistakes → squash commits, dentist in Berlin → how to call Germany
- However, simple tech guides can be brief and straightforward

### Titles

- How-to posts use imperative verbs: Susikurk, Suspausk, Nuspalvink, Pakeisk, Pasileisk, Surask, Užsidėk, Paskambink
- Other posts use descriptive nouns: "Docker sąvokos lietuviškai", "Draugiškos Python klasės", "Sprintas ar bintas?"

### Structure

- Research and plan before you act
- Be clear for every user and don't miss necessary steps for technical articles
- Explain the theory but also show practical examples
- Be concise, practical and close to the reader
- Use `###` headers for main sections
- Longer posts get an "Apibendrinimas" (summary) section near the end, before the literary quote
- For technical posts, include a "Šaltiniai" (Sources) section at the end — use bare links: `- Title URL`, not markdown `[Title](URL)`
- For German-related posts, include "Aktualus vokiškas žodynas" vocabulary section

### Literary Endings

End technical articles with a quote — from a book, film, interview, whatever fits. Separate it with `---`. Below are some of the authors and works that are cited in the blog.
- Matilda Olkinaitė
- Douglas Adams "Hitchhiker's Guide to the Galaxy"
- Dino Buzzati 
- Arūnas Žebriūnas

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

Then update the frontmatter if needed:

```
---
title: Short descriptive title
description: One-line summary if appropriate
date: YYYY-MM-DDTHH:MM:SSZ
author: Ąžuolas Krušna
tags: ["Tag1", "Tag2"]
ShowCodeCopyButtons: true  # For posts with code
draft: true
---
```

## Important Reminders

- PaperMod provides i18n translations — The theme includes `i18n/en.yaml` and `i18n/lt.yaml` with all standard UI translations (prev_page, next_page, read_time, toc, translations, home, edit_post, code_copy, etc.). Our custom `i18n/` files should only override specific translations or add custom ones (like "Posts"/"Pages"). Do NOT duplicate PaperMod's standard translations.
