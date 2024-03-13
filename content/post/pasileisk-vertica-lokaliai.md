---
title: Pasileisk Vertica lokaliai
date: 2024-03-13T03:00:00Z
ShowCodeCopyButtons: true
draft: true
---

Vertica labai nesunkiai galima pasileisti lokaliai! Kaip Docker konteinerį! Nors tai nėra labai populiarus sprendimas ir jo reikia paieškoti internete, bet jis aprašytas net oficialioje Vertica dokumentacijoje! Reikėtų paminėti, kad nemokama Vertica licenzija leidžia iki 1 terabaito duomenų (TB) bei iki 3 mazgų (nodes).

Pamenu, kad radau šią galimybę prieš daugiau nei 3 metus, gal jau ir 4 artėja? Tuomet ir daug mažiau informacijos apie tai buvo. Kodėl man tai prireikė? Dirbau su dideliais duomenų kiekiais ir turėjau labai prastą internetą namuose. Galimai tai buvo per COVID-19 pandemiją, nes iki jos darbas vyko iš ofiso. Dirbti su kodo pakeitimais ir juos testuoti tapo labai sudėtinga dirbant su daug duomenų bei lėtu internetu. Taip ir kilo ši idėja --- pasileisti Vertica lokaliai.

Sprendimas buvo neįtikėtinai geras --- viršijo mano lūkesčius. Ne tik kad duomenys nebekeliavo interneto laidais taip negaišindami manęs --- visas bendravimas vyko mano kompiuteryje, bet kartu tai buvo papildoma platforma išsitestuoti kaip atrodo duomenys pakeitus kodą prieš paleidžiant jį į produkciją. Čia norėčiau paminėti, kad Vertica dokumentacija, o kartu ir aš nerekomenduoju šio sprendimo taikyti produkcijai.

Taigi, viskas ką reikia, tai pasiimti šį "docker-compose.yml" [failą](https://github.com/AzisK/Vertica/blob/main/docker-compose.yml) arba jį taip apsirašyti.

```zsh
docker-compose.yml
```
```yml
version: '3.8'

services:
  vertica_ce:
    container_name: vertica_ce
    image: vertica/vertica-ce:10.1.1-0
    environment:
      - APP_DB_USER=vertica
      - APP_DB_PASSWORD=vertica
    ports:
        - 5433:5433
        - 5444:5444
    volumes:
      - type: volume
        source: vertica-data
        target: /data
volumes:
  vertica-data:
```

Žinoma, galima pasikeisti vartotojo vardą (APP_DB_USER) ir slaptažodį (APP_DB_PASSWORD), o kartu galima pasikeisti ir Vertica duomenų bazės versiją! Tuomet reikėtų nusirodyti kitą skaičiuką iš Docker registro "image" daly. Tačiau turint šį failą belieka pasikelti šį konteinerį su Docker kompozitoriumi 

```zsh
docker-compose up
```

ir jam sėkmingai pakilus galime jungtis į mūsų lokalią Vertica duomenų bazę savo kompiuteryje!


### Šaltiniai

- Vertica Community Edition (CE) https://www.vertica.com/docs/11.0.x/HTML/Content/Authoring/GettingStartedGuide/DownloadingAndStartingVM/DownloadingAndStartingVM.htm
- docker-compose.yml https://github.com/AzisK/Vertica/blob/main/docker-compose.yml
