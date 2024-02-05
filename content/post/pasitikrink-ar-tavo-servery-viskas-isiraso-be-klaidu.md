---
title: Pasitikrink, ar tavo servery viskas įsirašo be klaidų
description: Pasitikrink, ar tavo Python servery viskas įsirašo be klaidų naudojantis Docker
date: 2023-11-13T4:00:00Z
author: Ąžuolas Krušna
tags: ["Programų inžinerija", "Gidas", "Python", "Docker"]
ShowCodeCopyButtons: true
draft: true
---

Labas! Šiame straipsnyje dalinuosi kaip be didelio vargo pasitikrinti ar servery viskas įsirašo be klaidų. Tai nėra ypatingai sudėtinga teoriškai, bent jau žinant, kokius žingsnius reikia atlikti nuo šviežio operacinės sistemos įdiegimo. Visgi mes taip nedarysime, nes šiuos veiksmus pakartoti reikalauja daug veiksmų, nepatogu, nes reikia prisijungti prie serverio, o automatizuoti sudėtinga. Taigi tikrinsime Docker konteineriuose. Čia galime tai greitai ištestuoti net ir keičiantis pageidaujamų programų ir jų bibliotekų kiekiui bei versijoms.

Šiam gidui sekti reikalingos bent jau minimalios žinios apie Docker ir Python.

### Motyvacija rašyti

Trumpa įžanga noriu pasidalinti kodėl šis straipsnis buvo parašytas. Viskas prasidėjo nuo tai, kad šiuo metu prižiūriu vieną atviro kodo Python biblioteką Github platformoje, pypika. Ten vienas žmogus turėjo problemų įsirašant šią biblioteką ant vieno iš AWS (Amazon Web Services) serverių, "m7i/m6i Intel x86/64". Pasidalino, kad serveryje operacinė biblioteka yra Ubuntu 22.04, įrašytas vienas naujesnių Python 3.11.5 ir jam nepavyksta įrašyti Python bibliotekos vectordb-bench, nes jai reikalinga pypika biblioteka, kurią nesigauna įrašyti.

Daugiau detalių apie pypika bibliotekos vartotojo problemą galima paskaityti čia https://github.com/kayak/pypika/issues/761.

Pirmiausia, man pačiam reikėjo prisiminti kaip naudotis Docker tokiems atvejams ištirti, nes jau seniai tai bedariau ir labai primiršau. Buvo įdomu tai panagrinėti, prisiminti ir, iš tiesų, pagilinti savo Docker žinias bei galimybes. Nutariau parašyti viso šio proceso gidą lietuviškai, kad užtvirtinčiau savo Docker žinias bei įgūdžius, kad ilgiau ir stipriau tai prisiminčiau, nes šios žinios yra labai naudingos. Dar tik rašydamas straipsnį stebiuosi kiek daug žinių suteikia dalinimasis, aiškus dalinimasis, pasigilinimas į detales, kurias šiaip jau paviršutiniškai prabėgčiau, neva daugmaž suprantu kas čia vyksta, bele veikia. Tobulėju ne tik iš techninės pusės, bet ir kaip rašytojas, pasakotojas, susidėlioju savo mintis ir moyvaciją, kaip žmogus, tvarkausi savo vidinėse bibliotekose.

Man taip pat labai smagu rašyti lietuviškai. Vien dėl to, kad programų inžinerijos gidų lietuviškai beveik neegzistuoja. Kartu stengiuosi juos padaryti kiek galiu įdomesnius ir naudingesnius. Man velniškai patinka rašyti lietuviškai!

### Docker konteinerių elegancija

Kaip ir anksčiau minėta, šiame straipsnyje mes netikrinsime bibliotekų įrašymo pačiame serveryje, tačiau tai padarysime Docker konteineryje. Docker konteineriuose galime turėti bet kokią Linux operacinę sistemą ir įdiegti viską, ką norime. Taigi, galime išbandyti įvairias programų kombinacijas, ar jos gali būti įrašytos, ar gali būti įrašytos kartu, ar netrūksta kažkokios programėlės, ar jos nesikerta. Tam patikrinti mums reikia apsirašyti Docker instrukcijas, pagal kurias bus "tapomas" Docker paveikslas (image). Jas aprašome `Dockerfile` faile. Mes galime pasinaudoti jau aprašytomis instrukcijomis iš pageidaujamo Docker paveikslo ir ant viršaus įrašyti savo pageidaujamas instrukcijas.

Docker yra "nutapęs" Docker paveikslus su įvairiomis Python versijomis. Mes tuo pasinaudosime. Taigi apsirašome `Dockerfile` Python versiją.

```Dockerfile
ARG PYTHON_VERSION=3.11.5
```

Tuomet pasinaudojame jau sukurtu Docker paveikslu ir pratęsiame jo instrukcijas.

```Dockerfile
FROM python:${PYTHON_VERSION} as base
```

Čia yra panaudotas PYTHON_VERSION kintamasis, taigi šią eilutę galima suprasti kaip

```Dockerfile
FROM python:3.11.5 as base
```

Tuomet apsirašom 2 aplinkos kintamuosius, kurie yra naudingi Python aplikacijoms Docker konteineriuose leisti.

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

Tuomet dar atnaujiname `pip` biblioteką, kuri yra atsakinga už kitų bibliotekų įrašymą.

```Dockerfile
RUN pip3 install --upgrade pip
```

Tada įrašome Docker paveiksle bibliotekas iš `requirements.txt` failo. Papildoma komanda `--mount=type=cache,target=/root/.cache/pip` nurodom Docker išsaugoti parsiųstas Python bibliotekas kaip kešą ir panaudoti per naują jas siunčiantis ir kuriant šį Docker paveikslą.

```Dockerfile
RUN --mount=type=cache,target=/root/.cache/pip \
    pip3 install -r requirements.txt
```

Jau kaip ir viską turime, tačiau dar įkelsime į Docker paveikslą aplikacijos kodą, kurį savo direktorijoj laikysime `src` direktorijoje, o Docker paveiksle sukelsime į `app/src` direktoriją.

```Dockerfile
COPY src ./src
```

Galiausiai aprašome komandą, kurią iškviesime pagal nutylėjimą, jeigu iškviesime konteinerį, kuris pastatytas iš šio Docker paveikslo. Mūsų aplikacija bus iškviečiame `app.py` failu.

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

Dabar aprašysim Python programą, kuri mums atspausdins svarbiausius Docker paveikslo ir konteinerio parametrus. Tai, išties, labai paprasta programa, kuri pasisveikins, pasakys, kad visos Python bibliotekos sėkmingai įrašytos ir atspausdins mums Python versiją, pip bibliotekos versiją bei operacinės sistemos versiją. Į pačią programą čia nesigilinsime.

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

Docker konteinerio paleidimui supaprastinti apsirašysime `docker-compose.yaml` failą. Jame apsirašysime `dackeri` servisą, kuris naudos paveikslą, aprašytą iš `Dockerfile`. Tai apsirašo 4 eilutėmis

```yaml
services:
  dackeri:
    build:
      context: .
```

Dar prirašysime `command` nuorodą, kuria galėsime perrašyti `Dockerfile` esančią komandą ir ją paprastai perduoti. Docker paveikslą pavadinsime `dackeri`.  Galiausiai mūsų `docker-compose.yaml` atrodys taip

```bash
#docker-compose.yaml
```
```yaml
services:
  dackeri:
    build:
      context: .
    command: python3 src/app.py
```

Sukurti paveikslą iš Dockerfile galima žemiau aprašyta komanda.

```bash
docker-compose build
```

Paleisti konteinerį galim įvairiais būdais.

i) `docker-compose up` paleis visus `docker-compose.yaml` aprašytus servisus (šiuo atveju tik 1)

ii) `docker-compose up dackeri` paleis tik `dackeri` servisą

iii) `docker-compose run dackeri python3 src/app.py` paleis tam pačiam paveikslui tą pačią komandą, tačiau šis būdas yra patogus, nes mes komandine eilute galime pakeisti perduodamą komandą. Ji gali padėti suprasti bei debuginti Docker paveikslą, konteinerį. Pavyzdžiui, norėdami žinoti, kur esame konteinery tik į jį atėje, galime perduoti `pwd` komandą.

```bash
docker-compose run dackeri pwd
```
```bash
#Atsakymas
/app
```

Norėdami sužinoti šioje vietoje esančius failus ir direktorijas galime perduoti `ls` komandą.

```bash
docker-compose run dackeri ls
```
```bash
#Atsakymas
requirements.txt  src
```

Norėdami sužinoti Python versiją galime perduoti `python3 --version` komandą

```bash
docker-compose run dackeri python3 --version
```
```bash
#Atsakymas
Python 3.11.5
```

ir visas kitas Linux komandas.

Visų atvejų i), ii) ir iii) rezultastas yra vienodas

```bash
Labas, pasauli!
Python and its libraries have been successfully installed.
Python Version: 3.11.5 (main, Sep 20 2023, 21:16:49) [GCC 12.2.0]
pip Version: pip 23.3 from /usr/local/lib/python3.11/site-packages/pip (python 3.11)
Operating System: Linux 6.3.13-linuxkit
```

Valio! Pavyko įrašyti visas Python bibliotekas! Šis kodas nesunkiai gali būti pritaikytas kitai Python versijai ištestuoti pakeičiant `PYTHON_VERSION` kintamąjį `Dockerfile` faile. Kartu galime pakeisti pageidaujamas Python bibliotekas `requirements.txt` faile. Tada mums iš naujo reikia "nutapyti" Docker paveikslą `docker-compose build` komanda ir jeigu viskas sėkminga, tuomet leidžiame `docker-compose up` patikrinti kas buvo įrašyta ir naudojama. Rašau naudojama, nes gali būti įrašytos kelios Python versijos serveryje, bet naudojama ne ta, kurios mums reikia. Dar viena komanda, kuri gali būti naudinga, tai pamatyti iš kurios vietos Python komanda yra būtent naudojama.

```bash
docker-compose run dackeri which python3
```
```bash
#Atsakymas
/usr/local/bin/python3
```

Taip pat galime keisti (prirašyti, o gal neįrašyti) instrukcijas `Dockerfile`, jei tai reikalinga.


### Man "nenutapo" Docker paveikslo

Ką daryti, kai dėl kilusių klaidų paveikslas negali būti nutapomas? Jeigu būtume iš pradžių panaudoję `3.11.5-slim` Docker paveikslą kaip bazę, mums nebūtų pavykę įrašyti pageidaujamas bibliotekas. Pabandom.

4-ą `Dockerfile` eilutę pakeičiame šia

```Dockerfile
FROM python:${PYTHON_VERSION}-slim as base
```

Tada paleidę Docker paveikslo "tapymą" `docer-compose build` ties 4-u žingsniu įrašinėjant Python bibliotekas iš PyPI gauname šią didžiulę klaidą:

```bash
  Building wheel for psutil (pyproject.toml): started
  Building wheel for psutil (pyproject.toml): finished with status 'error'
  error: subprocess-exited-with-error

  × Building wheel for psutil (pyproject.toml) did not run successfully.
  │ exit code: 1
  ╰─> [45 lines of output]
      running bdist_wheel
      running build
      running build_py
      creating build
      creating build/lib.linux-aarch64-cpython-311
      creating build/lib.linux-aarch64-cpython-311/psutil
      copying psutil/_psbsd.py -> build/lib.linux-aarch64-cpython-311/psutil
      copying psutil/_psaix.py -> build/lib.linux-aarch64-cpython-311/psutil
      copying psutil/_pssunos.py -> build/lib.linux-aarch64-cpython-311/psutil
      copying psutil/_common.py -> build/lib.linux-aarch64-cpython-311/psutil
      copying psutil/_pslinux.py -> build/lib.linux-aarch64-cpython-311/psutil
      copying psutil/_psposix.py -> build/lib.linux-aarch64-cpython-311/psutil
      copying psutil/_compat.py -> build/lib.linux-aarch64-cpython-311/psutil
      copying psutil/_psosx.py -> build/lib.linux-aarch64-cpython-311/psutil
      copying psutil/_pswindows.py -> build/lib.linux-aarch64-cpython-311/psutil
      copying psutil/__init__.py -> build/lib.linux-aarch64-cpython-311/psutil
      creating build/lib.linux-aarch64-cpython-311/psutil/tests
      copying psutil/tests/test_memleaks.py -> build/lib.linux-aarch64-cpython-311/psutil/tests
      copying psutil/tests/test_misc.py -> build/lib.linux-aarch64-cpython-311/psutil/tests
      copying psutil/tests/test_sunos.py -> build/lib.linux-aarch64-cpython-311/psutil/tests
      copying psutil/tests/test_linux.py -> build/lib.linux-aarch64-cpython-311/psutil/tests
      copying psutil/tests/test_osx.py -> build/lib.linux-aarch64-cpython-311/psutil/tests
      copying psutil/tests/test_windows.py -> build/lib.linux-aarch64-cpython-311/psutil/tests
      copying psutil/tests/test_unicode.py -> build/lib.linux-aarch64-cpython-311/psutil/tests
      copying psutil/tests/runner.py -> build/lib.linux-aarch64-cpython-311/psutil/tests
      copying psutil/tests/__main__.py -> build/lib.linux-aarch64-cpython-311/psutil/tests
      copying psutil/tests/test_aix.py -> build/lib.linux-aarch64-cpython-311/psutil/tests
      copying psutil/tests/test_contracts.py -> build/lib.linux-aarch64-cpython-311/psutil/tests
      copying psutil/tests/test_bsd.py -> build/lib.linux-aarch64-cpython-311/psutil/tests
      copying psutil/tests/test_connections.py -> build/lib.linux-aarch64-cpython-311/psutil/tests
      copying psutil/tests/test_posix.py -> build/lib.linux-aarch64-cpython-311/psutil/tests
      copying psutil/tests/test_process.py -> build/lib.linux-aarch64-cpython-311/psutil/tests
      copying psutil/tests/test_system.py -> build/lib.linux-aarch64-cpython-311/psutil/tests
      copying psutil/tests/test_testutils.py -> build/lib.linux-aarch64-cpython-311/psutil/tests
      copying psutil/tests/__init__.py -> build/lib.linux-aarch64-cpython-311/psutil/tests
      warning: build_py: byte-compiling is disabled, skipping.

      running build_ext
      building 'psutil._psutil_linux' extension
      creating build/temp.linux-aarch64-cpython-311
      creating build/temp.linux-aarch64-cpython-311/psutil
      gcc -Wsign-compare -DNDEBUG -g -fwrapv -O3 -Wall -fPIC -DPSUTIL_POSIX=1 -DPSUTIL_SIZEOF_PID_T=4 -DPSUTIL_VERSION=596 -DPy_LIMITED_API=0x03060000 -DPSUTIL_ETHTOOL_MISSING_TYPES=1 -DPSUTIL_LINUX=1 -I/usr/local/include/python3.11 -c psutil/_psutil_common.c -o build/temp.linux-aarch64-cpython-311/psutil/_psutil_common.o
      psutil could not be installed from sources because gcc is not installed. Try running:
        sudo apt-get install gcc python3-dev
      error: command 'gcc' failed: No such file or directory
      [end of output]

  note: This error originates from a subprocess, and is likely not a problem with pip.
  ERROR: Failed building wheel for psutil
Successfully built pypika
Failed to build psutil
ERROR: Could not build wheels for psutil, which is required to install pyproject.toml-based projects
------
failed to solve: process "/bin/sh -c pip3 install -r requirements.txt" did not complete successfully: exit code: 1
```

Ši pranešimas yra labai ilgas, esminė informacija slypi 3-ose eilutėse

```bash
    psutil could not be installed from sources because gcc is not installed. Try running:
      sudo apt-get install gcc python3-dev
    error: command 'gcc' failed: No such file or directory
```

Mums trūksta `gcc` bibliotekos, kad galėtų būti įrašyta `psutils` biblioteka. Mes šia problemą galime išspręsti prieš `pip3 install -r requirements.txt` eilutę įrašę ją

```Dockerfile
RUN apt-get update && apt-get install gcc -y
```

Tuomet mūsų Docker paveikslas "nusitapo" be trikdžių.


### Velnias slypi detalėse

_Bus pratęsta_

***

*Bet man patinka rašyti. Tai esu supratęs. Ir skaitydamas atsimenu, kaip ten buvo. Ir skaitydamas atsimenu dar ir kai kuriuos dalykus, kuriuos buvau pamiršęs. Va tada man šauna į galvą, kad kartkarčiais reikia perskaityti tai, ką jau anksčiau esu skaitęs, ir varyti toliau. Tai šitaip aš perverčiau senąjį darbo kalendorių, kur rašiau anksčiau, ir radau šviesos greičio telefono numerį.*

Oskaras iš “Mane vadina Kalendorium”
