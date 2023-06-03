+++
title = "Suspausk savo git commits į vieną"
description = "Suspausk savo git commits į vieną imituojant squash merge"
date = 2023-06-03T03:00:00Z
author = "Ąžuolas Krušna"
+++

# Suspausk savo git commits į vieną

Angliškas terminas suspausti kelis _git_ commits į vieną vadinamas "squash merge" ir labai plačiai naudojamas.

Github ir Bitbucket platformos, o tikriausiai ir daug daugiau, turi šias galimybes pagal nutylėjimą, tačiau šios žinios vis tiek gali mums praversti. "Squash merge" gali būti neįjungtas, platforma gali tokios galimybės neturėti, mes galim visa tai padaryti be platformos pagalbos ir tai yra proga susipažinti su įvairiom _git_ funkcijomis.

Taigi pavadinkime savo kodo pakeitimų šaką "our-feature-branch". Taip jau nutiko, kad buvau pavargęs, pridariau klaidų, perskubėjau taisyti nepatikrinęs, todėl galiausiai prireikė 4 _git_ commits tam, kad turėčiau be priekaištų veikiančius pageidaujamus pakeitimus. Tos klaidų taisymo žinutės man tikrai nebus reikšmingos iš ateities perspektyvos, tik trukdys pamatyti kitas svarbias žinutes. Natūralu, kad jų matyti nenoriu. Čia ir suspausiu savo _git_ commits į vieną.

Šiam veiksmui atlikti susikursime tarpinę "temp" šaką. Veiksmų seka aprašyta žemiau. 
```bash
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

Nieko naujo inžinerijos pasaulyje nesugalvojau, bet pačiam prisireikė šios funkcijos, taigi išbandžiau, veikia, viskas labai paprasta ir genialu. Aprašiau visa tai lietuviškai ir perkošiau savo mintimis. Išminties sėmiausi iš šio StackOverflow [posto](https://stackoverflow.com/a/69827502/7714279).

Vienas žmogus net parašė bash [skriptą](https://github.com/sheerun/git-squash) visam šiam mechanizmui. Jį galima įsirašyti net su _brew_. Tai nėra visiškai šis skriptas, jis turi daugiau detalių, bet esmė yra ta pati --- suspausti savo _git_ šakoje commits į vieną.

***

_May the force be with you_
