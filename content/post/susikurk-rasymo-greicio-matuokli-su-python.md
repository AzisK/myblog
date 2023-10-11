+++
title = "Susikurk rašymo greičio matuoklį su Python"
description = "Susikurk rašymo greičio matuoklį per komandinę eilutę su Python"
date = 2023-10-06T02:13:50Z
author = "Ąžuolas Krušna"
tags = ["Programų inžinerija", "Susikurk savo", "Rašymo greitis", "Matuoklis", "Gidas", "Python", "Software", "Create your own", "Writing speed", "Speedometer", "Tutorial"]
+++

Labas! Dalinuosi smagiu pavyzdžiu kaip pačiam pasirašyti rašymo greičio matuoklį su Python. Šios dirbtuvės skirtos visiems, norintiems praplėsti savo Python žinias prie įdomaus projekto. Manau, kad jos bus įdomios tiek pradedantiems, tiek pažengusiems programuotojams, nes gilinamasi į neįprastus funkcionalumus. 

Kursime rašymo greičio matuoklį su Python. Jeigu mėgsti iššūkius ir norėtum su šia idėja daugiau pagalvoti savarankiškai, tuomet siūlau užsirašyti šią idėją, užverti šį puslapį ir linkiu džiugių atradimų! O jeigu neturi labai daug laiko ir norėtum, kad šis gidas tave palydėtų, tuomet jis laukia tavęs žemiau!

Rašymo greičio matuoklį kursime per komandinę eilutę. Angliškai tokios aplikacijos trumpinamos CLI raidėmis (command line application). Pirmiausia apsibrėžiame programos veikimą. Pamenu, kad pradinėse klasėse būdavo nuimamas laikas ir reikėdavo perskaityti kuo daugiau žodžių per vieną minutę. Greitį matuosime analogiškai, pagal parašytą kiekį pateiktų žodžių per minutę. Techniškai paprasčiau būtų išmatuoti vidutinį žodžių kiekį per minutę, bet mes lengvo kelio nesirenkame, o ir jis neatitiktų pradinių klasių lietuvių kalbos skaitymo greičio matavimų.

Taigi, mums būtina sąlyga yra gauti atsakymą (įvestį, _input_) iš rašytojo. Python paprastai tai gaunama naudojantis `input` funkcija.

```python
text = input()
```

Tačiau mums šis sprendimas netinka dėl trijų priežasčių.

i) Rašytojas turi paspausti "Enter", kad gauti iš jo atsakymą

ii) Ši funkcija yra blokuojanti, todėl kol ji laukia atsakymo iš rašytojo, niekas kitas programoje nevyksta

iii) Ja neįmanoma suskaičiuoti parašytų žodžių kiekio praėjus minutei, nes i) ir ii)


_May the force be with you_
