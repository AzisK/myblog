---
title: "Kintamieji YAML faile"
description: Kaip panaudoti kintamuosius YAML faile ir nekopijuoti tų pačių eilučių
date: 2026-06-27T15:00:00Z
author: Ąžuolas Krušna
tags: [Programų inžinerija, YAML, DevOps, Docker, GitLab, Kubernetes, CI/CD]
ShowCodeCopyButtons: true
---

> #### pi; n
>
> - `&vardas` sukuria kintamąjį
> - `*vardas` įklijuoja kintamajį
> - `<<: *vardas` sujungia kintamojo bloko raktus su lokaliais (lokalūs nugali)

```yaml
x-bendra: &bendra
  restart: always

services:
  web:
    <<: *bendra
    image: nginx
```

Labas! Šiandien sužinosime kaip apsirašyti ir panaudoti kintamuosius YAML failuose! Viskas labai paprasta, mums tereikia sužinoti kaip panaudoti 3 ar 4 simbolius! Ar mums prireiks 4-o, priklauso nuo aplinkos, tačiau bet kuriuo atveju susipažinsime.

## Kokiais simboliais yra aprašomi kintamieji?

Kintamiesiems apsirašyti ir panaudoti yra naudojami 3 pagrindiniai simboliai: `&`, `*`, `<<`. Tuomet priklausomai nuo aplinkos gali būti priekyje naudojamas `x-`, `.` ar kt.

### 1. `&` — inkaras (apibrėžia pakartojamą bloką)

Simbolis `&` sukuria inkarą, taigi žymeklį į tam tikrą bloką, kurį galime panaudoti kitur. 

```yml
x-bendra: &bendra
  restart: always
```

Taigi `bendra` yra mūsų kintamasis, kuris įsimena, kad po jo yra blokas `restart: always`. `x-bendra` gali būti pavadintas bet kaip.

### 2. `*` — alias (panaudoja įsimintą bloką)

Simbolis `*` yra alias, taigi perkelia inkaro pažymėtą bloką čia, nukopijuoja jį.

Apsirašome servisą ir konfigūracijai panaudojame apsirašytą `bendra` kintamąjį.

```yaml
services:
  config: *bendra
```

Po išskleidimo `service.config` tampa lygiai tuo, ką laikė `&bendra` kintamasis:

```yaml
services:
  config:
    restart: always
```

### 3. `<<` — sujungimo raktas (sulieja, o ne pakeičia)

Simbolis `<<`, kaip ir `*`, panaudoja mūsų aprašytąjį bloką su inkaru, tačiau tuo pačiu leidžia ir perrašyti tam tikrus laukus.

```yaml
services:
  web:
    <<: *bendra
    image: nginx
```

Po išskleidimo atrodys taip:

```yaml
services:
  web:
    restart: always
    image: nginx
```

Šalia parašytas laukas visuomet turės pirmumą, nesvarbu ar jį parašysime prieš sujungimo raktą `>>`, ar po.

```yaml
services:
  web:
    <<: *bendra
    image: nginx
  db:
    <<: *bendra
    restart: on-failure
    image: postgres
```

išsiskleidžia į šai tokią YAML konfigūraciją:

```yaml
services:
  web:
    restart: always
    image: nginx
  db:
    restart: on-failure
    image: postgres
```

### 4. `x-` ar `.` — plėtinio laukas (vieta šablonams laikyti)

Sužinojome kaip apsirašyti kintamuosius, juos panaudoti, tačiau kur juos dėti? Jei naudojame Docker, tuomet Docker mums neleis naudoti `bendra` kaip aukščiausio lygio rakto, nes tokio nepalaiko. Docker prirašę priekyje `x-` turime plėtinio lauką, kurį galime panaudoti kitur. Docker Compose jį sąmoningai praleidžia, todėl pačioje pradžioje apsirašėme taip:

```yaml
x-bendra: &bendra
  restart: always

services:
  web:
    <<: *common
    image: nginx
```

Tuo metu GitLab galime naudoti taško `.` simbolį priekyje. Tuomet šis darbo procesas taps paslėptu, tačiau bus galima jį perpanaudoti kitose vietose.

```yaml
.default_scripts: &default_scripts
  - ./default-script1.sh
  - ./default-script2.sh

job1:
  script:
    - *default_scripts
    - ./job-script.sh
```

## Kaip pasitikrinti?

Norėdami įsitikinti, ar teisingai viską apsirašėme, galime pasitikrinti. Yra keli būdai tai padaryti.

### 1. Docker Compose

Pasitikrinimui, ar teisingai viską apsirašėme, galime panaudoti `docker compose` `config` komandą. Tarkime, mūsų konfigūracija aprašyta `yaml.yml` faile. Tuomet tikriname taip:

```zsh
docker compose -f yaml.yml config
```

### 2. Python kodu

Kitas būdas pasitikrinti būtų naudojant Python. Importuojame `yaml` priklausomybę ir `safe_load` metodu nuskaitome YAML failą. Tuomet jį tiesiog atspausdiname.

```python
import yaml

with open("yaml.yml", "r") as f:
    data = yaml.safe_load(f)

print(yaml.safe_dump(data, sort_keys=False, allow_unicode=True))
```

Galime ir pasirašyti komandinės aplikacijos programėlę, kuriai galėsime perduoti norimus YAML failus:

```python
import sys

import yaml

path = sys.argv[1] if len(sys.argv) > 1 else "yaml.yml"

with open(path, "r") as f:
    data = yaml.safe_load(f)

print(yaml.safe_dump(data, sort_keys=False, allow_unicode=True))
```

### 3. Internete

Pasitikrinti galime ir internete šiuose adresuose:

- [yamllint.com](https://www.yamllint.com/)
- [onlineyamltools.com](https://onlineyamltools.com/)

## Svarbu žinoti

1. `<<` sulieja tik žemėlapius (mappings), ne sąrašus (lists)

Štai kodėl Kafka pavyzdyje kiekvienas brokeris turi savo `ports:` — sąrašų sujungti negalima.

2. Inkaras turi būti apibrėžtas prieš aliasą
 
YAML skaitomas iš viršaus į apačią, todėl kintamųjų blokus `x-` reikia apsirašyti pirmiau.

3. `<<` sujungimo rakto nėra oficialioje YAML 1.2 specifikacijoje

Tai populiarus papildymas (angl. _extension_). Praktiškai jį palaiko beveik visi įrankiai (Docker Compose, GitLab, dauguma bibliotekų), bet retas griežtas parser'is gali nepalaikyti.

---

Štai viskas! Dabar mokame apsirašyti kintaamuosius YAML failuose ir juos panaudoti bei netgi žinomem kaip tiksliai juos naudoti spcifinėse platformose kaip Docker Compose ir GitLab!

Inkarai ir alias yra pačios YAML kalbos dalis, todėl veikia visur, kur naudojamas YAML. Taigi ir GitHub, Kubernetes, Ansible, CircleCI ir kt.

---

_Aš Motė, sušnabždu į ausį Šventajam, kai visus apvaikščiojęs prisėda prie manęs, Lukai, įsivaizduoji, Motė, jis palinguoja galva, neblogai, sako, ot, būtume porelė, jei ne tas loterijos bilietas, išsišiepia. Gal tu čia mane ir užkrėtei, juokiuosi, šūdžiau tu, su visom savo galiom."_ 

Čiauškėjo Ona Kotrynos Zylės romane "Mano mylimi kaulai"

---

### Šaltiniai

- YAML Anchors and Aliases https://github.com/yaml/yaml-grammar/blob/master/yaml-spec-1.2.txt
- Docker Compose extension fields https://docs.docker.com/reference/compose-file/fragments/
- GitLab CI YAML anchors https://docs.gitlab.com/ci/yaml/yaml_optimization/#anchors
