---
title: Atsinaujink pre-commit priklausomybes paprastai
description: Kaip automatiškai atnaujinti versijas .pre-commit-config.yaml faile ir nesukti sau galvos
date: 2026-06-29T08:00:00Z
author: Ąžuolas Krušna
tags: ["Programinės įrangos inžinerija", "Python", "pre-commit", "prek"]
ShowCodeCopyButtons: true
draft: true
---
> #### pi; n
> `pre-commit autoupdate`\
> arba\
> `prek auto-update`

`.pre-commit-config.yaml` faile esantys kabliai (angl. _hooks_) gali lengvai pasenti, tad kaipgi juos paprastai atsinaujinti?

Labai paprastai, tiesiog šia komanda:

```bash
pre-commit autoupdate
```

Jeigu naudoji prek:

```bash
prek auto-update
```

Vienintelis skirtumas toks, kad prek naudoja `auto-update`, o ne `autoupdate`.

Tarkime, kad turime tokius kablius su savo versijomis (rev) pre-commit konfigūracijos faile:

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

Paleidus atsinaujinimą matysime kažką panašaus:

```
[https://github.com/astral-sh/ruff-pre-commit] updating v0.11.0 -> v0.15.14
[https://github.com/pre-commit/pre-commit-hooks] updating v4.3.0 -> v6.0.0
[https://github.com/pre-commit/mirrors-mypy] updating v1.10.0 -> v2.1.0
```

prek netgi turi saugumo sumetimais funkciją neįrašyti pačių naujausių versijų. Tereikia perduoti argumentą `--cooldown-days` su norima reikšme, pavyzdžiui:

```bash
prek auto-update --cooldown-days 14
```

Štai ir viskas. Gero kodo!

Tai atlikę, galime suformuoti commit:

```bash
git add .pre-commit-config.yaml
git commit -m "Atnaujinti pre-commit kablių versijas"
```

### Šaltiniai

- pre-commit https://pre-commit.com/
- prek https://github.com/j178/prek
