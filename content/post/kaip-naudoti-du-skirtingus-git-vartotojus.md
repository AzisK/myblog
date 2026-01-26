---
title: Kaip naudoti du skirtingus git vartotojus
description: Kaip naudoti du skirtingus git vartotojus skirtingose direktorijose
date: 2026-01-26T03:00:00Z
author: Ąžuolas Krušna
tags: ["Programų inžinerija", "Git"]
ShowCodeCopyButtons: true
---

Labas! Tai galima pasiekti susikonfigūruojant kiekvieną repozitoriją atskirai, tačiau galima tai sukonfigūruoti ir pagal direktorijas.

### Atskirai

Kiekvienai repozitorijai galime nustatyti vartotojo vardą ir el. paštą `git config` komanda.

```zsh
cd ~/personal/personal-project
git config user.name "Vardas Pavardė"
git config user.email "your@personal.email"
```

### Pagal direktoriją

Arba galime tai sukonfigūruoti naudodami `includeIf` sąlygą. Pavyzdžiui, darbo vartotojo vardą bei el. paštą naudojame globaliai, o asmeniniams naudojame atskirą konfigūraciją. Pavyzdžiui, visoms repozitorijoms `~/personal/` direktorijoje.

Tuomet `~/.gitconfig` faile apsirašome globalaus vartotoją vardą bei el. paštą. Kartu pridedame ir `includeIf` sąlygą, kurioje nurodome naudoti kitą konfigūracijos failą `.gitconfig-personal` repozitorijoms `~/personal/` direktorijoje.

```ini
[user]
    name = Vardas Pavardė
    email = your@work.email

[includeIf "gitdir:~/personal/"]
    path = ~/.gitconfig-personal
```

O `~/.gitconfig-personal` faile:

```ini
[user]
    name = Vardas Pavardė
    email = your@personal.email
```

---

Štai ir viskas! Dabar visi projektai `~/personal/` direktorijoje naudos asmeninį el. paštą, o visi kiti --- darbo.

### Šaltiniai

- Git Configuration: Conditional includes https://git-scm.com/docs/git-config#_conditional_includes
