---
title: Pakeisk git repozitorijos nuotolinį adresą
description: Pakeisk git repozitorijos nuotolinį adresą (URL)
date: 2026-02-08T04:00:00Z
author: Ąžuolas Krušna
tags: ["Programų inžinerija", "Git", "GitHub", "GitLab", "SSH"]
ShowCodeCopyButtons: true
---

Labas! Šiandien pasikeisime git repozitorijos nuotolinį adresą (URL).

Tai padaryti labai paprasta su šia komanda:

```zsh
git remote set-url origin https://naujas-adresas.git
```

Mums tereikia pakeisti `https://naujas-adresas.git` į mūsų norimą repozitorijos adresą.

## Kaip sužinoti koks yra mūsų dabartinis repozitorijos adresas?

Tai galime pasitikrinti šia komanda:

```zsh
git remote -v
```

Rezultatas gali atrodyti taip:

```zsh
origin  https://github.com/senas-vartotojas/repozitorija.git (fetch)
origin  https://github.com/senas-vartotojas/repozitorija.git (push)
```

Tada pasinaudojame `git remote set-url` komanda, kad pakeistume adresą:

```zsh
git remote set-url origin https://github.com/naujas-vartotojas/repozitorija.git
```

Galiausiai patikriname ar adresas pasikeitė:

```zsh
git remote -v
```

Rezultatas:

```zsh
origin  https://github.com/naujas-vartotojas/repozitorija.git (fetch)
origin  https://github.com/naujas-vartotojas/repozitorija.git (push)
```

## O jei naudoju SSH?

Jeigu naudoji SSH, procesas toks pat, tik adresas atrodo kitaip:

```zsh
git remote set-url origin git@github.com:vartotojas/repozitorija.git
```

### Kai naudoji kelis SSH raktus

Jeigu naudoji [du skirtingus SSH raktus](/kaip-naudoti-du-skirtingus-ssh-raktus/) (pvz., asmeninį ir darbo), tada adrese naudoji savo _Host alias_ vietoj `github.com`:

```zsh
git remote set-url origin git@personal.github.com:vartotojas/repozitorija.git
```

## Kada to gali prireikti?

- Pasikeitus repozitorijos talpinimo vietai
- Perkeliant repozitoriją į kitą paskyrą ar organizaciją
- Keičiant protokolą iš HTTPS į SSH (ar atvirkščiai)
- Susikonfigūravus [skirtingus SSH raktus](/kaip-naudoti-du-skirtingus-ssh-raktus/) skirtingoms paskyroms

---

Štai ir viskas! Pakeisti nuotolinį adresą --- *easy peasy, lemon squeezy*.

## Šaltiniai

- How do I change the URI (URL) for a remote Git repository? https://stackoverflow.com/a/2432799/7714279
- Changing a remote's URL https://docs.github.com/en/get-started/getting-started-with-git/managing-remote-repositories
