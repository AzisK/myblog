---
title: Docker sąvokos lietuviškai
description: Docker sąvokų apsibrėžimas lietuviškai bei jų tarpusavio ryšys
date: 2023-10-25T04:00:00Z
author: "Ąžuolas Krušna"
tags: ["Programų inžinerija", "Lietuvių kalba", "Konteineris", "Atvaizdas", "Docker"]
ShowCodeCopyButtons: true
---

Labas! Šiame straipsnyje lietuviškai apsibrėšime bei įsivardinsime pagrindines Docker platformos sąvokos. Šis straipsnis nėra skirtas supažindinti su Docker platforma lietuviškai, nors ir gali suteikti minimalų suvokimą.

Kodėl jis apskritai reikalingas? Lietuviškai kalbėti ir rašyti apie programinės įrangos inžineriją nėra lengva, nes visi terminai yra kilę iš anglakalbio pasaulio, todėl prieš gilinantis labai naudinga apsibrėžti svarbiausias sąvokas. Taip gimė ir šis trumpas straipsniukas, nes naudoju Docker rašydamas kitą straipsnį ir manau, kad dar ne vienoj vietoj naudosiu, todėl bus naudingas ateity tiek man, tiek skaitytojui. Kartu internete neradau šaltinių lietuviškai, kurie apie tai išsamiai rašytų.

### Pagrindinės Docker sąvokos

#### i) Docker failas (Dockerfile) ir jo instrukcijos

Docker failas yra sudarytas iš instrukcijų, kurias Docker atliks, kad sukurtų mūsų pageidaujamą sistemą. Įprastai šis failas yra pavadinamas `Dockerfile` vardu.

Docker failo pavyzdys
```bash
#Dockerfile
```
```Dockerfile
FROM python:3.11.5-slim as base
WORKDIR /app
COPY requirements.txt .
RUN pip3 install --upgrade pip
RUN pip3 install -r requirements.txt
COPY app.py .
CMD python3 app.py
```

#### ii) Docker atvaizdas (Docker image)

Nurodžius Docker įvykdyti instrukcijas, esančias Docker faile, kuriamas sistemos atvaizdas. Sėkmingai atlikus visas instrukcijas, ši izoliuota failų sistema yra išsaugojama. Tai ir yra Docker atvaizdas --- specifiškai paruošta failų sistema. Šį atvaizdą dabar bet kuriuo metu galime panaudoti pakeliant konteinerius --- paleidžiant šias paruoštas failų sistemas.

Docker atvaizdo kūrimo komanda:
```bash
docker build [OPTIONS] PATH | URL | -
```

Komandos naudojimo pavyzdžiai

a) Docker atvaizdo kūrimas naudojant lokalų Docker failą (`Dockerfile`)
```bash
docker build .
```

b) Docker atvaizdo kūrimas naudojant Docker failą (`Dockerfile`) iš Github repozitorijos
```bash
docker build github.com/creack/docker-firefox
```

Žemiau, 1 paveikslėlyje, galime matyti galimą Docker atvaizde išsaugotą failų sistemą.

![Docker](../docker_image_file_system.png)
1 pav. Galima Docker atvaizdo failų sistema
#### iii) Docker konteineris (Docker container)

Galime paleisti šį atvaizdą suktis suteikiant šiai sistemai resursų. Taip turėsime konteinerį, kuris įvykdys komandas ir išsijungs (pavyzdžiui, ištestuos, suskaičiuos duomenis ir padės į duomenų bazę ir pnš.) arba suksis tol, iki kol nebus sustadytas (duomenų bazės, internetinių puslapių serveriai ir pnš.). Iš vieno Docker sistemos atvaizdo galime sukurti daugelį Docker konteinerių.

Docker konteinerio paleidimo komanda:
```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

Komandos naudojimo pavyzdžiai

a) Interaktyvus Debian Docker atvaizdo paleidimas
```bash
docker run -it debian
```

b) Python aplikacijos Docker atvaizdo (`dackeri-dackerisimple`) paleidimas kartu su Python komanda `python3 src/app.simple.py`
```bash
docker run dackeri-dackerisimple python3 src/app.simple.py
```

#### iv) Docker kompozitorius (Docker Compose)

Docker kompozitorius yra įrankis, skirtas automatizuoti Docker konteinerius. Jis naudoja jau aprašytą Docker kompozicijos konfigūraciją paleidžiant konteinerius iš atvaizdų bei perduodant papildomus parametrus.

Pagrindinės Docker kompozitoriaus komandos:
- `docker-compose build` skirta perkurti kompozicijos konfigūracijoje aprašytus atvaizdus.

Ši komanda automatizuoja visų konfigūracijoje esančių Docker atvaizdų perrašymą vietoj `docker build ...` komandų.

- `docker-compose up` skirta pakelti konteinerius pagal konfigūraciją.

Ši komanda automatizuoja visų konfigūracijoje aprašytų konteinerių pakėlimą vietoj `docker run ...` komandų.

#### v) Docker kompozicijos konfigūracija (compose.yaml configuration)

Docker kompozicijos konfigūracija aprašo servisus, pagal kuriuos bus kuriami Docker konteineriai. Servisai naudoja Docker atvaizdas, tačiau gali perduoti ir papildomus parametrus. Vienas iš dažniausiai perduodamų parametrų gali būti vykdymo komanda `command`. Įprastai konfigūracija yra saugoma `compose.yaml` faile YAML formatu. Žemiau pateiktas paprastas Docker konfigūracijos pavyzdys iš vienos Github repozitorijos.

```bash
#compose.yaml
```
```yaml
services:
  dackeri:
    build:
      context: .
    command: python3 src/app.py
  dackerisimple:
    build:
      context: .
      dockerfile: Dockerfile.simple
    command: python3 src/app.simple.py
```

Ši konfigūracija `dackeri` servisui naudoja Docker atvaizdą, kuriamą pagal Docker failą iš esamos direktorijos (`.`) pagal nutylėjimą (`Dockerfile`). `dackerisimple` servisui yra naudojamas atvaizdas, sukurtas pagal `Dockerfile.simple` Docker failą. Taip viena komanda (`docker-compose up`) yra pakeliami 2 Docker konteineriai, naudojantys 2 skirtingus Docker atvaizdus bei vykdantys 2 skirtingas komandas.

***

Žemiau esantis 2 paveikslėlis vaizduoja ryšį tarp Docker failo, atvaizdo ir konteinerio.

![Docker](../docker.jpg)
2 pav. Ryšys tarp tarp Docker failo, atvaizdo ir konteinerio

3 paveikslėly matome ryšį tarp pagrindinių Docker savokų. Šis paveikslėlis turi papildomą Docker failą, Docker atvaizdą ir konteinerį, kurie yra įtraukti į Docker kompozicijos konfigūraciją ir gali būti koordinuojami kartu.

![Docker](../docker_compose.jpg)
3 pav. Ryšys pagrindinių Docker savokų

### Apibendrinimas

Apžvelgėme pagrindines Docker sąvokas ir jas apsibrėžėme lietuviškai. Tai padės mums suprasti bei rašyti Docker platformą naudojančius straipsnius! Kartu susipažinome su jų naudojimo pavyzdžiais. Tai padės vėliau naudojantis Docker platforma.

***

**P.S.** Turint galimybę automatizuoti konteinerių paleidimą bei atvaizdų įrašymą, asmeniškai naudoju beveik vien tik Docker kompozitoriaus `docker-compose` komandas ir beveik nenaudoju grynų `docker build` ir `docker run` komandų, nes visą logiką galiu aprašyti `compose.yml` faile, tad neberašyti įvairių parametrų komandinėje eilutėje ir, turbūt svarbiausia, galiu nebeatsiminti tų visų parametrų. Kartu galiu išsaugoti visą grupę naudojamų vaizdų bei prikelti visą grupę norimų konteinerių.

***

**P.P.S.** Parašiau beveik visą gana ilgą straipsnį naudodamas terminą "Docker paveikslas", kuris yra daug vaizdingesnis, tačiau vėliau pakeičiau į "Docker atvaizdą", nes šį variantą radau literatūroje internete, kartu ir dėl to, kad paveikslo tapymas yra nepakartojamas menas, o inžinerijoje dėmesio verti tik pakartojami reiškiniai. Docker atvaizdas privalo kiekvieną kartą veikti vienodai patikimai visoms sistemoms ir jų vartotojams, kitaip negalėsime juo pasikliauti ir naudotis.

***

*Perskaičiau viską kelis kartus. Man tie tekstai vis tiek patinka. Aš neketinu ką nors keisti. Gal jie nesupranta? Šiaip ar taip, kad aktoriams nepatinka, yra mano kaltė, bet ne teksto kaltė. Nesu su jais pakankamai apie tai kalbėjęs. Bet čia nėra ko stebėtis. Aš labai mažai kalbu. Čia šuo ir pakastas. Tai rytoj aš jiems ir pasakysiu, kodėl mažai kalbu.  Dabar man tai aišku.*

Oskaras iš "Mane vadina Kalendorium".

***

### Šaltiniai

- docker build https://docs.docker.com/engine/reference/commandline/build/
- docker run https://docs.docker.com/engine/reference/commandline/run/
- Docker Compose overview https://docs.docker.com/compose/
- Overview of the get started guide https://docs.docker.com/get-started/
