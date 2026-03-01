---
title: Kaip naudoti du skirtingus git vartotojus
description: Kaip naudoti du skirtingus git vartotojus skirtingose direktorijose
date: 2026-01-26T03:00:00Z
author: Ąžuolas Krušna
tags: ["Programų inžinerija", "Git", "Github", "GitLab", "SSH"]
ShowCodeCopyButtons: true
---

Labas. Nusirodyti git vartotojo vardą ir el. paštą galime kiekvienoje repozitorijoje atskirai. Tačiau galime ir tai automatiškai susikonfigūruoti pagal direktorijas.

### Atskirai

Kiekvienai repozitorijai galime nustatyti vartotojo vardą ir el. paštą `git config` komanda.

```zsh
cd ~/personal/personal-project
git config user.name "Name Surname"
git config user.email "your@personal.email"
```

### Pagal direktoriją

Galime tai susikonfigūruoti `~/.gitconfig` faile naudodami `includeIf` sąlygą. Pavyzdžiui, šiame faile apsirašome globaliai naudojamą git vartotojo vardą, el. paštą ir įtraukiame salygą naudoti kitą konfigūraciją tam tikroj direktorijoj esančioms repozitorijoms --- mūsų atveju `~/personal/`. Nurodome, kad naudoti šį konfigūracijos failą --- `~/.gitconfig-personal`.

Tuomet mūsų failai gali atrodyti taip:

```ini
<!-- ~/.gitconfig -->
[user]
    name = Name1 Surname1
    email = your@work.email

[includeIf "gitdir:~/personal/"]
    path = ~/.gitconfig-personal
```

```ini
<!-- ~/.gitconfig-personal -->
[user]
    name = Name2 Surname2
    email = your@personal.email
```

---

Štai ir viskas! Dabar visi projektai `~/personal/` direktorijoje naudos asmeninį el. paštą, o visi kiti --- darbo.

---

_Bus..._

_Visai netikėtai\
Šį rytą, šį rytą,\
Visai netikėtai\
Jau snaigės iškrito._

_Koks džiaugsmas, kiek laimės!\
Ir pinasi dainos --\
Kalėdos, Kalėdos --\
Atostogos eina!_

_Bus ledas, bus sniegas,\
Bus raštai ant lango.\
Kalėdos, Kalėdos!\
Atostogos brangios._

_Bus Liuncė, bus Liuncė\
Ir bus rogių kelias,\
Bus daug vakaruškų\
Ir bus kaukių balius._

_Nevieriai ir Bimbos,\
Klišiai ir Kavoliai,\
Šilkinėm skarelėm\
Sulėks Pan'munėlin_

_Bus raštai ant lango,\
Bus sniegas, bus ledas --\
Bus Liuncė, bus Liuncė!\
O brangios Kalėdos!_

1938 m. parašytas Matildos Olkinaitės eilėraštis apie ateinančią sniego ir ledo žiemą, apie grožį ir džiaugsmą, kurį ji atneša, apie Liuncę, galimai jos geriausią draugę. 

Ši žiema Lietuvoje buvo šalta ir graži kaip jau seniai bebūta, rekordai minėjo apie 14 metų. Gaila, kad šią žiemą nebuvau Lietuvoje, iš tiesų, buvau šiek tiek, Kūčių naktį parvažiavau, per Kalėdas, iki Naujų Metų, bet buvo labai per mažai (refleksiją rašau kovo 1-ąją, pirmą pvasario dieną).

---

### Šaltiniai

- Git Configuration: Conditional includes https://git-scm.com/docs/git-config#_conditional_includes
