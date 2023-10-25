+++
title = "Pasitikrink, ar tavo servery viskas įsirašo be klaidų"
description = "Pasitikrink, ar tavo Python servery viskas įsirašo be klaidų naudojantis Docker"
date = 2023-11-13T04:00:00Z
author = "Ąžuolas Krušna"
tags = ["Programų inžinerija", "Gidas", "Python", "Docker", "Software", "Tutorial"]
+++

Labas! Šiame straipsnyje dalinuosi kaip be didelio vargo pasitikrinti ar servery viskas įsirašo be klaidų. Tai nėra ypatingai sudėtinga teoriškai, bent jau žinant, kokius žingsnius reikia atlikti nuo šviežio operacinės sistemos įdiegimo. Visgi mes taip nedarysime, nes šiuos veiksmus pakartuoti reikalauja daug veiksmų, nepatogu, nes reikia prisijungti prie serverio, o automatizuoti sudėtinga. Taigi tikrinsime Docker konteineriuose. Čia galime tai greitai ištestuoti net ir keičiantis pageidaujamų programų ir jų bibliotekų kiekiui bei versijoms.

Šiam gidui sekti reikalingos bent jau minimalios žinios apie Docker.

Trumpa įžanga noriu pasidalinti kodėl šis straipsnis buvo parašytas. Viskas prasidėjo nuo tai, kad šiuo metu prižiūriu vieną atviro kodo Python biblioteką Github platformoje, pypika. Ten vienas žmogus turėjo problemų įsirašant šią biblioteką ant vieno iš AWS (Amazon Web Services) serverių, "m7i/m6i Intel x86/64". Pasidalino, kad serveryje operacinė biblioteka yra Ubuntu 22.04, įrašytas vienas naujesnių Python 3.11.5 ir jam nepavyksta įrašyti Python bibliotekos vectordb-bench, nes jai reikalinga pypika biblioteka, kurią nesigauna įrašyti.

Daugiau detalių apie pypika bibliotekos vartotojo problemą galima paskaityti čia https://github.com/kayak/pypika/issues/761.

Pirmiausia, man pačiam reikėjo prisiminti kaip naudotis Docker tokiems atvejams ištirti, nes jau seniai tai bedariau ir labai primiršau. Buvo įdomu tai panagrinėti, prisiminti ir, iš tiesų, pagilinti savo Docker žinias bei galimybes. Nutariau parašyti viso šio proceso gidą lietuviškai, kad užtvirtinčiau savo Docker žinias bei įgūdžius, kad ilgiau ir stipriau tai prisiminčiau, nes šios žinios yra labai naudingos. Dar tik rašydamas straipsnį stebiuosi kiek daug žinių suteikia dalinimasis, aiškus dalinimasis, pasigilinimas į detales, kurias šiaip jau paviršutiniškai prabėgčiau, neva daugmaž suprantu kas čia vyksta, bele veikia. Tobulėju ne tik iš techninės pusės, bet ir kaip rašytojas, pasakotojas, susidėlioju savo mintis ir moyvaciją, kaip žmogus, tvarkausi savo vidinėse bibliotekose.

Man taip pat labai smagu rašyti lietuviškai. Vien dėl to, kad programų inžinerijos gidų lietuviškai beveik neegzistuoja. Kartu stengiuosi juos padaryti kiek galiu įdomesnius ir naudingesnius. Man velniškai patinka rašyti lietuviškai!

### Docker konteinerių elegancija

Kaip ir anksčiau minėta, šiame straipsnyje mes netikrinsime bibliotekų įrašymo pačiame serveryje, tačiau tai padarysime Docker konteineryje. Docker konteineriuose galime turėti bet kokią Linux operacinę sistemą ir įdiegti viską, ką norime. Taigi, galime išbandyti įvairias programų kombinacijas, ar jos gali būti įrašytos, ar gali būti įrašytos kartu, ar netrūksta kažkokios programėlės, ar jos nesikerta. Tam patikrinti mums reikia apsirašyti Docker paveikslą (image). Jį aprašome `Dockerfile` faile. Mes galime pasinaudota jau aprašytu pageidaujamu Docker paveikslu ir ant viršaus įrašyti pageidaujamas funkcijas.

Docker yra sukūręs Docker paveikslus su įvairiomis Python versijomis. Mes tuo pasinaudosime. Taigi apsirašome `Dockerfile` Python versiją.

```Dockerfile
ARG PYTHON_VERSION=3.11.5
```

Tuomet pasinaudojame jau sukurtu Docker paveikslu ir jį pratęsiame.

```Dockerfile
FROM python:${PYTHON_VERSION} as base
```

Čia yra panaudotas PYTHON_VERSION kintamasis, taigi šią eilutę galima supasti kaip 

```Dockerfile
FROM python:3.11.5 as base
```

Tuomet apsirašom 2 aplinkos kintamuosius, kurie yra naudingi Python aplikacijoms Docker konteineriuose. 

`PYTHONDONTWRITEBYTECODE=1` nustatymas nurodo Python nerašyti baitinės programos į failus, t.y. į `.pyc` failus. Tai yra naudinga tik tuomet, kai Python programa paleidžiama daugiau nei 1 kartą, o konteineriuose tai mums nesuteiks jokių privalumų, tik apsunkins viską. 

`PYTHONUNBUFFERED=1` nustatymas nurodo Python nesaugoti informacijos buferyje, tai padeda išvengti logų vėlavimo.

```Dockerfile
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
```

Paskui pereiname į `/app` direktoriją, kaip yra įprasta Python aplikacijoms Docker konteineriuose.

```Dockerfile
WORKDIR /app
```

Dabar mums reikia susikurti `requirements.txt` failą, kurį vėliau perkelsime į Docker paveikslą. Taigi susikuriam tokį, įrašom jame pageidaujamas bibliotekas ir grįžtame redaguoti `Dockerfile`. Šiuo atveju mūsų pageidaujamos bibliotekos yra nurodytos žemiau

```bash
#requirements.txt
```
```bash
pypika
vectordb-bench
```

Tuomet perkeliame šį `requirements.txt` failą į Docker paveikslą šia eilute

```Dockerfile
COPY requirements.txt .
```

Galutinė `requirements.txt` vieta paveiksle bus `/app/requirements.txt`, nes mes prieš tai jau nukeliavom į `/app` direktoriją.

Tuomet dar atnaujima `pip` biblioteką, kuri yra atsakinga už kitų bibliotekų įrašymą.

```Dockerfile
RUN pip3 install --upgrade pip
```

Tada įrašome Docker paveiksle bibliotekas iš `requirements.txt` failo. Papildoma komanda `--mount=type=cache,target=/root/.cache/pip` nurodom Docker išsaugoti parsiųstas Python bibliotekas kaip kašą ir panaudoti per naują jas siunčiantis ir kuriant šį Docker paveikslą.

```Dockerfile
RUN --mount=type=cache,target=/root/.cache/pip \
    pip3 install -r requirements.txt
```

Jau kaip ir viską turime, tačiau dar įkelsime į Docker paveikslą aplikacijos kodą, kurį savo direktorijoj laikysime `src` direktorijoje, o Docker paveiksle sukelsime į `app/src` direktoriją.

```Dockerfile
COPY src ./src
```

Galiausiai aprašome komandą, kurią iškviesime pagal nutylėjimą, jeigu iškviesime konteinerį, kuris pastatytas iš šio Docker paveikslo. Mūsų aplikaciją bus iškviečiame `app.py` failu.

```Dockerfile
CMD python3 src/app.py
```

Šauniai padirbėjom, dabar galime pažiūrėti kaip galutinai atrodo mūsų `Dockerfile`.
```bash
#Dockerfile
```
```Dockerfile
# Define desired Python version number
ARG PYTHON_VERSION=3.11.5
# Use an official Python image as a base
FROM python:${PYTHON_VERSION} as base

# Prevents Python from writing pyc files.
ENV PYTHONDONTWRITEBYTECODE=1

# Keeps Python from buffering stdout and stderr to avoid situations where
# the application crashes without emitting any logs due to buffering.
ENV PYTHONUNBUFFERED=1

# Move to /app directory
WORKDIR /app

# Copy requirements.txt into the image
COPY requirements.txt .

# Upgrade pip
RUN pip3 install --upgrade pip

# Download Python dependencies as a separate step to take advantage of Docker's caching
# Leverage a cache mount to /root/.cache/pip to speed up subsequent builds
# Install Python dependecies
RUN --mount=type=cache,target=/root/.cache/pip \
    pip3 install -r requirements.txt

# Copy the source code into the container.
COPY src ./src

# Run the application.
CMD python3 src/app.py
```

Dabar aprašysim Python programą, kuri mums atspausdins svarbiausius Docker paveikslo ir konteinerio parametrus. Tai, išties, labai paprasta programa, kuri pasisveikins, pasakys, kad visos Python bibliotekos sėkmingai įrašytos ir atspausdins mums Python versiją, pip bibliotekos versiją bei operacinės sistemos versiją. Į pačią programą dabar nesigilinsime.

```bash
#app.py
```
```python
import sys
import subprocess
import platform


# Hello world in Lithuanian
print('Labas, pasauli!')
print('Python and its libraries have been successfully installed.')
print('Python Version:', sys.version)

try:
    pip_version = subprocess.check_output([sys.executable, '-m', 'pip', '--version']).decode('utf-8')
    print(f'pip Version: {pip_version.strip()}')
except subprocess.CalledProcessError as e:
    print('pip is not installed or not found.')

# Print OS information
print('Operating System:', platform.system(), platform.release())
```

***

_May the force be with you_
