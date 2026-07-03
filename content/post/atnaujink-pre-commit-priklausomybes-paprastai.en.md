---
title: Update pre-commit Dependencies the Easy Way
slug: update-pre-commit-hook-versions
description: How to automatically update versions in your .pre-commit-config.yaml file without overthinking it
date: 2026-07-03T08:00:00Z
author: Ąžuolas Krušna
tags: ["Software Engineering", "Python", "pre-commit", "prek"]
ShowCodeCopyButtons: true
draft: true
---

> #### tl; dr
> `pre-commit autoupdate`\
> or\
> `prek auto-update`

The hooks in `.pre-commit-config.yaml` can easily go stale, so how do you update them the simple way?

Very simply, with this command:

```bash
pre-commit autoupdate
```

If you use prek:

```bash
prek auto-update
```

The only difference is that prek uses `auto-update` instead of `autoupdate`.

Say we have hooks with pinned versions (`rev`) in the pre-commit config file:

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.11.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.3.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.10.0
    hooks:
      - id: mypy
        additional_dependencies: ["types-tqdm"]
```

After running the update, you’ll see something like this:


```text
[https://github.com/astral-sh/ruff-pre-commit] updating v0.11.0 -> v0.15.14
[https://github.com/pre-commit/pre-commit-hooks] updating v4.3.0 -> v6.0.0
[https://github.com/pre-commit/mirrors-mypy] updating v1.10.0 -> v2.1.0
```

prek even has a safety feature that avoids writing the very newest versions right away. Just pass the `--cooldown-days` argument with the value you want, for example:

```bash
prek auto-update --cooldown-days 14
```

That’s it. Happy coding!

After that, we can make a commit:

```bash
git add .pre-commit-config.yaml
git commit -m "Update pre-commit hook versions"
```

### Sources

- pre-commit https://pre-commit.com/
- prek https://github.com/j178/prek
