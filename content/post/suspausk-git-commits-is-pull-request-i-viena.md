---
title: "Suspausk git commits iš pull-request į vieną"
description: "Suspausk git commits iš pull-request į vieną panaudojant tarpinę šaką"
date: 2023-06-13T03:00:00Z
author: "Ąžuolas Krušna"
tags: ["Programų inžinerija", "Git"]
---

Angliškas terminas suspausti kelis git commits iš pull-request į vieną vadinamas "squash (and) merge" ir labai plačiai naudojamas.

Taip jau nutiko, kad buvau pavargęs, pridariau klaidų, perskubėjau taisyti nepatikrinęs, todėl galiausiai prireikė 4 git commits tam, kad turėčiau be priekaištų veikiančius pageidaujamus pakeitimus. Tos klaidų taisymo žinutės man tikrai nebus reikšmingos iš ateities perspektyvos, tik trukdys pamatyti kitas svarbias žinutes. Natūralu, kad jų matyti nenoriu. Čia ir suspausiu savo git commits į vieną.

Github ir Bitbucket platformos turi šią galimybę pagal nutylėjimą. Žemiau galime matyti kaip atrodo pull-request suspaudimo ir prijungimo (squash and merge) galimybė Github platformoje.

![Squash merge demonstration](../squash_merge_demonstration.png)

Jeigu mums tiek ir tereikėjo, tuomet užverčiame šį puslapį ir einam pasimėgauti likusia diena!

### Kaip tai padaryti naudojantis terminalu?

Gali nutikti taip, kaip nutiko man, kad kodo versijavimo platforma prarado šią galimybę. Taigi "squash (and) merge" gali būti neįjungtas arba platforma gali tokios galimybės neturėti.

Tuomet mes pasinaudosime git komandomis commits suspaudimui į vieną ir galėsim visa tai padaryti net ir be jokios platformos pagalbos! Tapsime nepriklausomi nuo git kodo versijavimo sistemų. Kartu tai yra proga labiau susipažinti su įvairiomis git komandomis.

Taigi, mūsų pull-request šaka vadinasi "our-feature-branch". Žemiau aprašyti 3 (A, B ir C) būdai jos pokyčius prijungti su 1 commit.

#### A. Suspaudžiame ir prijungiame pull-request šakos pokyčius

Esame ant pagrindinės "main" šakos.

```zsh
git checkout main
```

Tuomet visus skirtumus nuo mūsų pasirinktos šakos sujungiame į vieną commit ir prijungiame.

```zsh
git merge --squash our-feature-branch
```

Šiuo atveju turime tvarkingą pagrindinę šaką, tačiau mūsų pull-request beveik praranda savo esmę. Dabar patikrinę šios šakos pull-request nematysime jokių pokyčių ir nebebus ką prijungti (merge). Tokiu atveju belieka uždaryti (close) šį pull-request.

#### B. Suspaudžiame commits mūsų pull-request šakoj į vieną

Tai yra aprašyta ankstesniame straipsnyje ["Suspausk savo git commits į vieną"](https://www.aziogas.lt/suspausk-savo-git-commits-i-viena). Tuomet galime prijungti pull-request paprastuoju būdu (merge).

Šis būdas yra geras, tačiau jam reikia atsiminti, pasirinkti, suskaičiuoti commits, kuriuos mes norime suspausti į vieną. Žemiau aprašytas būdas turi privalumą, kad mums beveik nereikia galvoti. Jis automatiškai visus šakos pokyčius suspaudžia į vieną commit.

#### C. Suspaudžiame commits mūsų pull-request šakoj į vieną pasinaudojant pagalbine šaka

Šiam veiksmui atlikti panaudosime tarpinę "temp" šaką. Veiksmų seka aprašyta žemiau.
```zsh
git checkout -b temp main #1

git merge --squash our-feature-branch #2

git commit "OUR BIG MESSAGE" #3

git checkout our-feature-branch #4

git reset --hard temp #5

git push --force #6

git branch -d temp #7
```

1. Atskeliame kodo šaką "temp" nuo pagrindinės mūsų kodo šakos "main"
2. Suspaudžiame visus kodo pakeitimus nuo "main" iš "our-feature-branch" į "temp" šaką
3. Užrašome šių kodo pakeitimų žinutę į "temp" šaką
4. Keliaujame į "our-feature-branch" šaką
5. Perrašom ją su visais pakeitimais, kuriuos turi "temp" šaka, taigi tą vieną vienintelį, kur visi pakeitimai suspausti į 1 commit
6. "Jėga" (force) perrašome "our-feature-branch" šaką ir mūsų nuotolinėje repozitorijoje
7. Ištriname "temp" šaką

Jėė! Dabar niekas nebematys mano "Bug fix" commits.

Dabar galime daryti mūsų pull-request paprastą prijungimą (merge).

***

Šiam straipsniui parašyti buvo remtasi šiuo [Stack Overflow](https://stackoverflow.com/a/69827502/7714279) įrašu bei paties atradimais. Mano nuomone, C būdas yra genialus, bet labai geras ir B.

***

Vienas žmogus net parašė bash [skriptą](https://github.com/sheerun/git-squash) visam šiam mechanizmui. Jį galima įsirašyti net su _brew_. Tai nėra visiškai šis skriptas, jis turi daugiau detalių, bet esmė yra ta pati --- suspausti savo git šakoje commits į vieną, panaudojant tarpinę šaką.

_May the force be with you_
