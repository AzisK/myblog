+++
title = "Suspausk git commits iš pull-request į vieną"
description = "Suspausk git commits iš pull-request į vieną panaudojant tarpinę šaką"
date = 2023-06-13T03:00:00Z
author = "Ąžuolas Krušna"
+++

Angliškas terminas suspausti kelis git commits iš pull-request į vieną vadinamas "squash (and) merge" ir labai plačiai naudojamas.

Github ir Bitbucket platformos turi šią galimybę pagal nutylėjimą. Žemiau galime matyti kaip atrodo pull-request suspaudimo ir prijungimo (squash and merge) galimybė Github platformoje.

![Squash merge demonstration](../squash_merge_demonstration.png)

Jeigu mums tiek ir tereikėjo, tuomet užverčiame šį puslapį ir einam pasimėgauti likusia diena.

### Kaip tai padaryti naudojantis terminalu?

Tačiau gali nutikti taip, kaip nutiko man, kad kodo versijavimo platforma prarado šią galimybę. Taigi "squash merge" gali būti neįjungtas arba platforma gali tokios galimybės neturėti.

Tuomet mes naudosime git CLI commits suspaudimui į vieną ir galėsim visa tai padaryti net ir be jokios platformos pagalbos! Tapsime nepriklausomi nuo git kodo versijavimo sistemų. Kartu tai yra proga giliau susipažinti su įvairiomis git komandomis.

Žemiau galime rasti 3 būdus tai atlikti.

#### A. Suspaudžiame ir prijungiame pasirinktos šakos pokyčius

Esam ant pagrindinės `main` šakos. Tuomet visus skirtumus nuo mūsų pasirinktos (`our-feature-branch`) šakos sujungiame į vieną commit ir prijungiame.

```zsh
git merge --squash our-feature-branch
```

Šiuo atveju turime tvarkingą pagrindinę šaką, tačiau mūsų pull-request beveik praranda savo esmę. Dabar patikrinę šios šakos pull-request nematysime jokių pokyčių ir nebebus ką prijungti (merge). Tokiu atveju belieka uždaryti (close) šį pull-request.

#### B. Suspaudžiame mūsų pull-request šakoj commits į vieną

Tai yra aprašyta ankstesniame straipsnyje ["Suspausk savo git commits į vieną"](https://www.aziogas.lt/suspausk-savo-git-commits-i-viena). Tuomet galime prijungti pull-request paprastuoju būdu (merge).

Šis būdas yra geras, tačiau jam reikia atsiminti, pasirinkti, suskaičiuoti commits, kuriuos mes norime suspausti į vieną. Žemiau aprašytas būdas turi privalumą, kad mums beveik nereikia galvoti. Jis automatiškai visus šakos pokyčius suspaudžia į vieną commit.

#### C. Suspaudžiame mūsų pull-request šakoj commits į vieną pasinaudojant pagalbine šaka

Pavadiname savo kodo pakeitimų šaką "our-feature-branch". Taip jau nutiko, kad buvau pavargęs, pridariau klaidų, perskubėjau taisyti nepatikrinęs, todėl galiausiai prireikė 4 git commits tam, kad turėčiau be priekaištų veikiančius pageidaujamus pakeitimus. Tos klaidų taisymo žinutės man tikrai nebus reikšmingos iš ateities perspektyvos, tik trukdys pamatyti kitas svarbias žinutes. Natūralu, kad jų matyti nenoriu. Čia ir suspausiu savo git commits į vieną.

Šiam veiksmui atlikti susikursime tarpinę "temp" šaką. Veiksmų seka aprašyta žemiau.
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

Nieko naujo inžinerijos pasaulyje nesugalvojau, bet pačiam prisireikė šios funkcijos, taigi išbandžiau, veikia, viskas labai paprasta ir genialu. Aprašiau visa tai lietuviškai ir perkošiau savo mintimis. Išminties sėmiausi iš šio StackOverflow [posto](https://stackoverflow.com/a/69827502/7714279).

***

Vienas žmogus net parašė bash [skriptą](https://github.com/sheerun/git-squash) visam šiam mechanizmui. Jį galima įsirašyti net su _brew_. Tai nėra visiškai šis skriptas, jis turi daugiau detalių, bet esmė yra ta pati --- suspausti savo git šakoje commits į vieną panaudojant tarpinę šaką.

_May the force be with you_
