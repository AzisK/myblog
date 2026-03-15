---
title: "Kaip kurti naujus įrašus su Hugo"
date: 2026-03-14T21:30:47+01:00
author: Ąžuolas Krušna
tags: ["Hugo", "Blogas", "Tinklaraštis"]
---

> #### pi; n
> hugo new post/kaip-kurti-naujus-irasus-su-hugo.md

Naujus įrašus galime kurti kuriant naujus `.md` failus atitinkamoje Hugo tinklaraščio repozitorijos direktorijoje, tačiau paprasčiau yra jau esant tinklaraščio repozitorijoje naudotis `hugo new` komanda.

## Komanda sukurti naują įrašą

Sukurti naują įrašą su hugo komanda galime taip:

```bash
hugo new content/kelias/iki/iraso/iraso-pavadinimas.md
```

Mano blogo struktūros atveju būtų taip:

```bash
hugo new content/post/kaip-kurti-naujus-irasus-su-hugo.md
```

Tačiau galima ir sutrumpinti:

```bash
hugo new post/kaip-kurti-naujus-irasus-su-hugo.md
```

Tai sukuria naują tinklaraščio failą su _frontmatter_ laukais.

Įrašo turinį galime patikrinti `cat` komanda:

```bash
cat content/post/kaip-kurti-naujus-irasus-su-hugo.md
```

## Naujo įrašo šablonas

Pats naujo įrašo sukūrimo šablonas yra aprašytas `archetypes/default.md` faile. Ten yra įrašyta tik _frontmatter_ dalies logika, nors galime apsirašyti ką tik panorėtume. Maniškis `archetypes/default.md` atrodo taip:

```bash
---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
author: 
tags: []
draft: true
---
```

Kaip matome, šablonas sukuria naują juodraštį (_angl_. draft), taigi šis įrašas nebus publikuojamas kol nebus išjungtas `draft` laukas.

Mano _frontmatter_ atrodo taip:

```bash
---
title: "Kaip Kurti Naujus Irasus Su Hugo"
date: 2026-03-14T21:30:47+01:00
author: Ąžuolas Krušna
tags: []
draft: true
---
```

## Peržiūrėti naudojant lokalų serverį:

Lokaliai pasileidžiant hugo serverį galime pridėti `-D` parametrą, kuris sugeneruos įrašus ir iš juodraščių:
```bash
hugo serve -D
```

## Pridėti turinį

Pridėti turinį galime atsidarę šį failą savo pasirinktu redaktoriumi:

```bash
zed content/post/kaip-kurti-naujus-irasus-su-hugo.md
nvim content/post/kaip-kurti-naujus-irasus-su-hugo.md
subl content/post/kaip-kurti-naujus-irasus-su-hugo.md
code content/post/kaip-kurti-naujus-irasus-su-hugo.md
# etc.
```

Paveikslėlius galime pridėti nurodant kelią iki jų nuo įrašo. Kadangi mano tinklarašty paveikslėlių internetinis adresas yra https://aziogas.com/kaip-kurti-naujus-irasus-su-hugo, o paveikslėliai randasi taip pat tiesiai ant domeno, pvz. https://www.aziogas.lt/netlify-domain.png, tuomet galime sugrįžti (du taškai `..`) ant domeno ir nurodyti paveikslėlio failo pavadinimą.

Tuomet paveikslėlio užrašymas atrodys taip:

```bash
![alternatyvus tekstas](../netlify-domain.png)
```

Prieš publikuojant nepamirštame nuimti `draft: true` eilutės, kitaip įrašas nebus matomas.

Smagaus rašymo!

---

> Nuliai
>
> Jei prirašysim nulių,\
> Tai nuliai ir tebus:\
> Nors jų bus milijonai --\
> Turėsim tik nulius.
>
> Bet vienetą pridėję\
> Prie nulių iš pradžių,\
> Mes gausim milijonus\
> Tada iš nulių tų.
>
> Ir tarp žmonių mes matom\
> Tų nulių daug labai,\
> Bet vienetų pasauly\
> Užtinkame retai.

1936 m. rašė Matildė Olkinatė

---

## Šaltiniai

1. A beginner's guide to making new posts with Hugo https://merrimanlab.github.io/post/2020-10-15-making-posts/
2. Creating new Hugo blog posts https://srdan.geek.nz/posts/friday-2nd-august-2024/
3. Create new post using Hugo https://pitchumanisivan.in/posts/hugo-new-post/
