---
title: Lengvai surask tai, ką jau rašei terminale
description: Lengvai surask tai, ką jau rašei terminale, naudojantis komandų istorija
date: 2023-12-20T04:00:00Z
author: Ąžuolas Krušna
tags: ["Programų inžinerija", "Terminalas", "Komandinė eilutė", "Istorija", "Atmintis", "Linux", "Software", "Terminal", "Command line", "History", "Memory"]
ShowCodeCopyButtons: true
---

*Šis gidas yra skirtas Apple gamybos Mac kompiuteriams*

Labas. Jeigu nori lengvai surasti tai, ką jau rašei terminale, bet nežinai kaip, tai šis gidas yra skirtas kaip tik tau!

Viskas labai paprasta --- tereikia rašyti "history", tarpą, vadinamąjį vamzdžio ženklą "|", tarpą bei paieškos frazę.

```zsh
history | grep <SEARCH>
```

Naudojimosi pavyzdys:

```zsh
history | grep python
```

Noriu sumažinti rezultatų kiekį iki 3, tuomet galiu "grep" komandai perduoti "-m" arba "\--max-count" kintamąjį su 3 reikšme.

```zsh
history | grep -m 3 python
```

Rezultatas:

```zsh
63  python manage.py shell
87  python --version
88  python3 --version
```

Jė! Dabar galėsiu nesunkiai rasti ką jau rašiau. Kartais pagaunu save situacijoje, kai man šis funkcionalumas labai praverstų, bet man trūksta žinių kaip jį pasiekti. Taigi dabar šios bėdos nebeturėsiu! 

Jeigu nori sužinoti daugiau --- skaityki toliau. Jeigu tiek ir reikėjo, tuomet meski tolesnį skaitymą be sąžinės graužaties.

### Alias!

Parašyti ir "history", ir "vamzdį", ir "grep" ir užtrunka, ir iš pradžių gali pasirodyti daug atsiminti, todėl visuomet galime susikurti alias! Atsidarom savo terminalo aplinkos failą ir ten apsirašom alias, taigi trumpinį. Mano tai atveju tai failai `~/.zprofile` arba `~/.zshrc`:

```zsh
alias hs="history | grep"
```

Gerai, dabar man reikės atsiminti tik "hs"! Ieškau jau rašytų terminale komandų su raktažodžiu "redis":

```zsh
hs redis -m 2
```

Atsakymas:

```zsh
65  redis-server /usr/local/etc/redis.conf
66  brew install redis
```

### Kokiais atvejais tai gali praversti?

Tai vieną dieną gali labai palengvinti darbus --- kai jie bus intensyvūs komandinėje eilutėje arba grįžę iš atostogų pamiršime kaip ten tiksliai naudotis "ta" komandinės eilutės aplikacija ir kokius ten jai argumentus perdavėm ir dar kokia tvarka.

### Kaip visa tai veikia?

Mūsų įrašų terminale istorija yra išsaugoma automatiškai. Ji saugoma `~/.bash_history` arba `~/.zshrc_history` failuose atitinkamai kokią terminalo aplinką naudojame. Visų mūsų įrašų eilutės iš šio failo ir yra pasiekiamos "history" komanda.

Patikrinti istorijos dydį galime išspausdinę "HISTSIZE" kintamąjį:

```zsh
echo $HISTSIZE
```

Mano atveju šio kintamojo reikšmė yra 50000.

Norėdami pakeisti istorijos dydį galime šį kintamąjį perrašyti savo terminalo aplinkos faile. Mano atveju tai `~/.zshrc` arba `~/.zprofile` failai:


```zsh
export HISTSIZE=100000
```

### O kaip išvalyti istoriją?

Norint išvalyti istoriją galime ištrinti istorijos failą bei išvalyti laikiną istoriją perdavę "history" komandai "c" argumentą. Mano atveju ši komanda atrodo taip:

```zsh
rm ~/.zshrc_history && history -c
```

### Apribendrinimas

Išmokome lengvai surasti tai, ką jau rašėme terminale anksčiau! Tam pasinaudojome 2 programomis --- "history" ir "grep". Apsirašėm toms komandoms trumpinį (alias). Taip pat sužinojom kaip tai veikia, išmokome pakeisti istorijos dydį bei ją išvalyti. Kartu apžvelgėme atvejus, kada tai gali būti naudinga.

***

#### *VAKARAS BE DARBO*

*Ateina vakaras ir pradeda palangėj kaukti\
\
Ir ilgesį vadinti girgždančiu balsu.\
\
Ir nežinai, žmogus, ko čia belaukti,\
\
Kai melsvas ilgesys vaidenas po langu…\
\
Ateina vakaras ir nešasi mėnulį seną,\
\
Ir mirkčioja pro langą spinksanti žvaigždė.\
\
Už krosnies kažin kur svirplys gyvena,\
\
O po grindim – vikri maža pelė.\
\
Ateina ilgesys ir seną smuiką nešas\
\
Ir serenadą griežia po langu.\
\
Ir kaip tu tokį svečią pavarysi\
\
Kai jis toks liūdnas ir be to toks mandagus?\
\
Ir darosi savęs taip baisiai gaila,\
\
Kada palangėj griežia ilgesys.\
\
Ir rodos tau, kad viskas – baisiai kvaila\
\
Pelė, mėnulis ir svirplys.\
\
Ir rodos tau, kad tu patsai lyg sukvailėjai\
\
Ir net poetu pavirtai,\
\
Kada, mėnulį nužiūrėjęs,\
\
Kažko beprasmiai dūsavai.*

Iš Matildos Olkinaitės dienoraščio
***

### Šaltiniai

Šį labai naudingą *know-how* mane pamokė kolega ir šios džiaugsmingos progos įkvėptas parašiau šį trumpą gidą. Gidas paremtais labiausiai įvertintais StackOverflow ir AskUbuntu atsakymais bei Linux žinynu.

- How do I limit the number of results returned from grep? https://stackoverflow.com/a/5013198/7714279
- grep(1) — Linux manual page https://man7.org/linux/man-pages/man1/grep.1.html
- View history of commands run in terminal https://askubuntu.com/a/624850
- history(3) — Linux manual page https://man7.org/linux/man-pages/man3/history.3.html
