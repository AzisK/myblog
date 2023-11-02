+++
title = "Susikurk rašymo greičio matuoklį su Python"
description = "Susikurk rašymo greičio matuoklį per komandinę eilutę su Python"
date = 2023-10-27T04:00:00Z
author = "Ąžuolas Krušna"
tags = ["Programų inžinerija", "Susikurk savo", "Rašymo greitis", "Matuoklis", "Gidas", "Python", "Software", "Create your own", "Writing speed", "Speedometer", "Tutorial"]
+++

Labas! Dalinuosi smagiu pavyzdžiu kaip pačiam pasirašyti rašymo greičio matuoklį su Python. Šios dirbtuvės skirtos visiems, norintiems praplėsti savo Python žinias prie įdomaus projekto. Manau, kad jos bus įdomios tiek pradedantiems, tiek pažengusiems programuotojams, nes gilinsimės į neįprastus funkcionalumus.

Kursime rašymo greičio matuoklį su Python. Jeigu mėgsti iššūkius ir norėtum su šia idėja daugiau pagalvoti savarankiškai, tuomet siūlau užsirašyti šią idėją, užverti šį puslapį ir linkiu džiugių atradimų! O jeigu neturi labai daug laiko ir norėtum, kad šis gidas tave palydėtų, tuomet jis laukia tavęs žemiau!

Rašymo greičio matuoklį kursime per komandinę eilutę. Angliškai tokios aplikacijos trumpinamos CLI raidėmis (command line application). Pirmiausia apsibrėšime programos veikimą. 

### Įkvėpimas

Pamenu, kad pradinėse klasėse mokytoja nuspausdavo chronometro mygtuką žemyn ir tuomet skubėdavau skaityti žodžius, perskaityti kuo daugiau žodžių per vieną minutę. Įkvėptas šių prisiminimų bei sklandžios technikos, nutariau nieko ir per daug nekeisti. Tik skirtingai nuo skaitymo, matuosime rašymo greitį kompiuterio klaviatūra. Matuoti skaitymo greitį būtų daug sudėtingiau, nes reikėtų žodžių atpažinimo technologijos bei mikrofono prieigos, o rašymo greičiui ranka matuoti prireiktų prieigos prie vaizdo kameros bei rašto atpažinimo technologijos.                                                                         

### Programos apsibrėžimas

Greitį matuosime pagal parašytą kiekį pateiktų žodžių per minutę. Techniškai paprasčiau būtų išmatuoti vidutinį žodžių kiekį per minutę, bet mes lengvo kelio nesirenkame, o ir jis neatitiktų pradinių klasių lietuvių kalbos skaitymo greičio matavimų.

### Kimbame į darbus

Taigi, mums būtina sąlyga yra gauti atsakymą (įvestį, _input_) iš rašytojo. Python paprastai tai gaunama naudojantis `input` funkcija.

```python
text = input()
```

Tačiau mums šis sprendimas netinka dėl trijų susijusių priežasčių.

i) Rašytojas turi paspausti "Enter", kad gauti iš jo atsakymą

ii) Ši funkcija yra blokuojanti, todėl kol ji laukia atsakymo iš rašytojo, niekas kitas programoje nevyksta

iii) Ja neįmanoma suskaičiuoti parašytų žodžių kiekio praėjus minutei, nes i) ir ii)

#### MacOS gidas

Todėl naudosimės kitu būdu gauti įvestį iš vartotojo, t.y. `sys.stdin.read` funkcija. Iš jos galime gauti tik nurodytą simbolių kiekį, kai paspaudžiame "Enter", pavyzdžiui, 3 `sys.stdin.read(3)` arba priimti atsakymą tol, iki kol vartotojas paspaus Ctrl+D ant MacOS kompiuterio.

Su aprašytu funkcionalumu mes dar kol kas neturime galimybių sukurti rašymo mūsų rašymo greičio matuoklio.

Galime pasinaudoti `termios` biblioteka, kuria nustatysime, kad programa nelauktų "Enter" paspaudimo ir nuskaitytų nuspaustą raidę iš karto.

```python
import sys
import termios

fd = sys.stdin.fileno()
new_term = termios.tcgetattr(fd)
new_term[3] = (new_term[3] & ~termios.ICANON)
termios.tcsetattr(fd, termios.TCSAFLUSH, new_term)

letter = sys.stdin.read(1)

print(letter)
```

Paleidę šią programą bei paspaudę raidę "L", komandinėje eilutėje pamatysime dvi "L" raides ir programa išsijungs.

```bash
LL
```

Tuomet mes galime komandinei eilutei perduoti nustatymą, kuris nespausdintų paspaustų raidžių, kitaip tariant, išjungtume aidą (`termios.ECHO`).

```python
import sys
import termios

fd = sys.stdin.fileno()
new_term = termios.tcgetattr(fd)
new_term[3] = (new_term[3] & ~termios.ICANON & ~termios.ECHO)
termios.tcsetattr(fd, termios.TCSAFLUSH, new_term)

letter = sys.stdin.read(1)

print(letter)
```

Dabar programa nuspaudus "L" parašys mums vieną "L" raidę ir išsijungs.

```bash
L
```

Mums nereikia, kad programa išsijungtų bei kartu reikia priimti daugiau raidžių iš vartotojo, todėl įdedame viską į ciklą.

***

_May the force be with you_
