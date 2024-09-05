---
title: Pasileisk Vertica duomenų bazę lokaliai
description: Pasileisk Vertica duomenų bazę lokaliai ant Docker
date: 2024-09-05T18:00:00Z
ShowCodeCopyButtons: true
---

Labas! Šiandien sužinosime kaip pasileisti savo Vertica duomenų bazę! Šiam techniniam gidui sekti pravers minimalios žinios apie Docker, tačiau jos nėra būtinos. Pasiskaityti apie Docker galima mano anksčiau rašytame įraše "[Docker sąvokos lietuviškai](https://www.aziogas.lt/docker-savokos-lietuviskai/)".

Vertica pasileisti lokaliai galime labai nesunkiai! Kaip Docker konteinerį! Mums reikia tik 1 Docker kompozitoriaus failo "docker-compose.yml" ir vienos komandos "docker-compose up". 

Nors tai nėra labai populiarus sprendimas ir jo reikia paieškoti internete, bet jis aprašytas net oficialioje Vertica dokumentacijoje! Reikėtų paminėti, kad nemokama Vertica licenzija leidžia iki 1 terabaito duomenų (TB) bei iki 3 mazgų (nodes). Visgi Docker aplinkoje pasileisime tik 1 mazgo Vertica, nes oficialiai Vertica palaiko tik tokį sprendimą. Kelių mazgų sprendimą palaiko tik Kubernetes aplinkoje. Šis sprendimas kiek sudėtingesnis, todėl jo dabar neapžvelgsime.

### Kam man reikalinga Vertica duomenų bazė lokaliai?

Pamenu, kad radau šią galimybę prieš daugiau nei 3 metus, o gal jau ir 4? Tuomet ir daug mažiau informacijos apie tai buvo. Kodėl man tai prireikė? Dirbau su dideliais duomenų kiekiais ir turėjau labai prastą internetą namuose. Tai turėjo būti COVID-19 pandemijos metu, nes iki jos darbas vyko iš ofiso. Dirbti su kodo pakeitimais ir juos testuoti tapo labai sudėtinga dirbant su daug duomenų bei lėtu internetu. Taip ir kilo ši idėja --- pasileisti Vertica lokaliai.

Sprendimas buvo neįtikėtinai geras --- viršijo mano lūkesčius. Ne tik kad duomenys nebekeliavo elektromagnetinėmis bangomis iki modemo bei interneto laidais per serverius taip negaišindami manęs --- visas bendravimas vyko mano kompiuteryje, bet kartu tai buvo papildoma platforma išsitestuoti kaip atrodo duomenys pakeitus kodą prieš paleidžiant jį į produkciją. Čia norėčiau paminėti, kad Vertica dokumentacija, o kartu ir aš nerekomenduoju šio sprendimo taikyti produkcijai.

### Kaip tai padaryti?

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
      - vertica-data:/data
volumes:
  vertica-data:
```

Žinoma, galima pasikeisti vartotojo vardą (APP_DB_USER) ir slaptažodį (APP_DB_PASSWORD), o kartu galima pasikeisti ir Vertica duomenų bazės versiją! Tuomet reikėtų nusirodyti kitą skaičiuką iš Docker registro "image" daly. Dabartinį Docker atvaizdą “vertica/vertica-ce:10.1.1-0” galime pakeisti bet kuriuo naujesniu ar senesniu iš Docker registrų, pavyzdžiui, “24.1.0-0”. Tačiau turint šį failą belieka pasikelti šį konteinerį su Docker kompozitoriumi 

```zsh
docker-compose up
```

ir jam sėkmingai pakilus galime jungtis į mūsų lokalią Vertica duomenų bazę savo kompiuteryje!

Jeigu tiek ir reikėjo, sveikinu! Tu jau turi Vertica duomenų bazę savo kompiuteryje ir gali uždaryti šį puslapį. Jeigu įdomu sužinoti daugiau automatizavimo galimybių, siūlau skaityti toliau.

### Kaip galiu turėti pradinę duomenų bazę pakilus konteineriui?

Pirmiausia noriu paminėti, kad Vertica konteineris jau turi paruošęs mums pavyzdinę duomenų bazę "VMart" su keleta lentelių. Tačiau ši duomenų bazė tikriausiai netenkins mūsų poreikių ir mes norėsim pasikelti duomenų bazės struktūrą pagal savo reikalavimus.

Tai nesunkiai galim padaryti pridėdami kelias rašmenas (angl. _script_) bei vieną papildomą eilutę mūsų Docker kompozitoriaus "docker-compose.yml" faile. Šis Vertica atvaizdas yra sudėliotas taip, kad pakeliant duomenų bazę kartu paleistų ir visas rašmenas, esančias "/docker-entrypoint-initdb.d" direktorijoje ir besibaigiančias ".sh" ir ".sql".

Aiškumo dėlei susikursime pas save lokaliai direktoriją tokiu pačiu pavadinimu "docker-entrypoint-initdb.d". Tuomet sudėsim į ją norimas rašmenas. O tada pasinaudodami Docker kompozitoriaus talpos (angl. _volume_) skilties surišimu perkelsime ją į Docker konteinerį.

Dabar mūsų Docker kompozitoriaus failas atrodo štai taip.

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
      - vertica-data:/data
      - ./docker-entrypoint-initdb.d:/docker-entrypoint-initdb.d
volumes:
  vertica-data:
```

Tuomet beliko apsirašyti SQL rašmenas, kurias Vertica konteineris paleis kiekvieną kartą jį pakėlus.

Jei Vertica konteineris paleistų pradinės duomenų bazės rašmenas tik pirmą kartą kuriant duomenų bazę, tuomet jas apsirašyti naudočiau 2 SQL rašmenas (jos prasukamos abecėlės tvarka): "1-create_table.sql" ir "2-insert_into.sql". Visgi man šis sprendimas netinka, nes norėčiau iš pradžių jau turėti duomenų, tačiau nenoriu jų vis per naują sukelti pakėlus konteinerį.

### Pradinės duomenų bazės rašmenos

Todėl naudosiu Shell rašmeną, kuri patikrins ar egzistuoja viena lentelė, kuri indikuoja ar visos reikiamos lentelės jau sukurtos. Tebūnie šios lentelės pavadinimas "InitiateDatabase". Ar ši lentelė egzistuoja, tikrinsiu tokia SQL užklausa:

```sql
SELECT EXISTS (
  SELECT 1 FROM v_catalog.tables WHERE table_name = 'InitiateDatabase'
);
```

Jeigu šios lentelės nėra, tuomet ši rašmena, analogiškai esamai logikai, abėcėlės tvarka paleis visas rašmenas, kurios baigiasi ".vsql".

Šią Shell rašmeną pavadinsiu "initdb.sh", o ji atrodys taip.

```zsh
docker-entrypoint-initdb.d/initdb.sh
```
```zsh
#!/bin/bash

# Full path to the vsql binary with dbadmin user and VMart database
VSQL="/opt/vertica/bin/vsql -U dbadmin -d VMart"

# Function to check if the InitiateDatabase table exists
init_table_exists() {
    local query="SELECT EXISTS (SELECT 1 FROM v_catalog.tables WHERE table_name = 'InitiateDatabase');"
    local result=$($VSQL -c "$query" | grep -w "t")
    if [ -n "$result" ]; then
        return 0
    else
        return 1
    fi
}

# Check if the InitiateDatabase table exists
if ! init_table_exists; then
    echo "InitiateDatabase table does not exist. Initializing the database..."

    # Run all .vsql scripts in the current directory
    for sql_file in $(ls /docker-entrypoint-initdb.d/*.vsql | sort); do
        echo "Running $sql_file..."
        $VSQL -f "$sql_file" > /tmp/vsql_output.log 2>&1
        if [ $? -ne 0 ]; then
            echo "Error running $sql_file. Check /tmp/vsql_output.log for details."
        else
            echo "Successfully ran $sql_file"
        fi
    done
else
    echo "Database has already been initialized. Skipping the initialization scripts."
fi
```

Taigi, logikai atskirti naudosiu 2 VSQL rašmenas. Pirmoji, "1-create_table.vsql", turės visą kodą, kuris atsakingas sukurti lenteles bei aprašyti jų struktūras. Antroji, "2-insert_into.vsql", sukels mūsų norimus pradinius duomenis į šias lenteles. Papildomai, antroji rašmena turės kodą, kuris sukurs "InitiateDatabase" lentelę.

Tuomet mūsų "1-create_table.vsql" rašmena atrodys taip.

```zsh
docker-entrypoint-initdb.d./1-create_table.vsql
```
```sql
CREATE TABLE IF NOT EXISTS Fruits (
    id IDENTITY PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    fruit VARCHAR(50) NOT NULL CHECK (fruit IN ('Apple', 'Orange')), -- Enforcing enum-like behavior
    color VARCHAR(255) NOT NULL,
    weight FLOAT NOT NULL,
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS Basket (
    basket_id IDENTITY PRIMARY KEY,
    description VARCHAR(255),
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS BasketFruits (
    basket_fruit_id IDENTITY PRIMARY KEY,
    basket_id INT,
    fruit_id INT,
    FOREIGN KEY (basket_id) REFERENCES Basket(basket_id),
    FOREIGN KEY (fruit_id) REFERENCES Fruits(id),
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

O mūsų "2-insert_into.vsql" rašmena atrodys taip.

```zsh
docker-entrypoint-initdb.d/2-insert_into.vsql
```
```sql
BEGIN;

-- Insert data into Fruits table
INSERT INTO Fruits (name, fruit, color, weight) VALUES ('Granny Smith', 'Apple', 'Green', 150.5);
INSERT INTO Fruits (name, fruit, color, weight) VALUES ('Red Delicious', 'Apple', 'Red', 160.0);
INSERT INTO Fruits (name, fruit, color, weight) VALUES ('Golden Delicious', 'Apple', 'Yellow', 170.2);
INSERT INTO Fruits (name, fruit, color, weight) VALUES ('Navel', 'Orange', 'Orange', 180.3);
INSERT INTO Fruits (name, fruit, color, weight) VALUES ('Valencia', 'Orange', 'Orange', 190.1);
INSERT INTO Fruits (name, fruit, color, weight) VALUES ('Blood Orange', 'Orange', 'Red', 200.4);

-- Insert data into Basket table
INSERT INTO Basket (description) VALUES ('Basket 1');
INSERT INTO Basket (description) VALUES ('Basket 2');
INSERT INTO Basket (description) VALUES ('Basket 3');

-- Insert data into BasketFruits table
INSERT INTO BasketFruits (basket_id, fruit_id) VALUES (1, 1); -- Basket 1 contains Granny Smith
INSERT INTO BasketFruits (basket_id, fruit_id) VALUES (1, 1); -- Basket 1 contains another Granny Smith
INSERT INTO BasketFruits (basket_id, fruit_id) VALUES (1, 4); -- Basket 1 contains Navel
INSERT INTO BasketFruits (basket_id, fruit_id) VALUES (2, 2); -- Basket 2 contains Red Delicious
INSERT INTO BasketFruits (basket_id, fruit_id) VALUES (2, 5); -- Basket 2 contains Valencia
INSERT INTO BasketFruits (basket_id, fruit_id) VALUES (3, 3); -- Basket 3 contains Golden Delicious
INSERT INTO BasketFruits (basket_id, fruit_id) VALUES (3, 6); -- Basket 3 contains Blood Orange

-- Create InitiateDatabase table if it does not exist
CREATE TABLE IF NOT EXISTS InitiateDatabase (
    initialized BOOLEAN NOT NULL,
    create_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert into InitiateDatabase table to help identify whether all of the SQL commands have ran successfully
INSERT INTO InitiateDatabase (initialized) VALUES (TRUE);

COMMIT;
```

Matome, kad VSQL rašmeną įsukau tarp BEGIN ir COMMIT SQL komandų. Taip padariau, nes kitu atveju man neįvykdydavo paskutinių komandų. Tik įsukus tarp BEGIN ir COMMIT tapau užtikrintas, kad visos rašmenos atsiras duomenų bazėje.

#### Ką mes sukūrėm pradinėj duomenų bazėj?

Mes sukūrėme pavyzdinę normalizuotą duomenų bazę. Pirmiausia, mes sukūrėme 3 lenteles: `Fruits` (vaisiai), `Baskets` (krepšeliai) ir `BasketFruits` (krepšelvaisiai). `BasketFruits` sukūrėm tam, kad galėtume apjungti `Fruits` ir `Baskets` lenteles, t.y. vienas vaisius galėtų būti daugelyje krepšelių ir vienam krepšely galėtų būti daugelis vaisių. Tuomet šiose lentelėse sukūrėme keletą vaisių ir keletą krepšelių, o tada į krepšelius pridėliojome vaisių per krepšelvaisių lentelę. 

Norėdami sužinoti kiek vaisių turi kiekvienas krepšelis galime sužinoti šia SQL užklausa.

```sql
SELECT
    b.basket_id,
    COUNT(bf.basket_fruit_id) fruit_count
FROM
    Basket b
JOIN
    BasketFruits bf ON b.basket_id = bf.basket_id
GROUP BY b.basket_id;
```

---

Rezultatas

```zsh
| basket_id | fruit_count |
|-----------|-------------|
| 2         | 2           |
| 1         | 3           |
| 3         | 2           |
```

---

Norėdami sužinoti kiekvieną vaisių kiekviename krepšely galime pasinaudoti šia SQL užklausa.

```sql
SELECT
    b.basket_id,
    b.description AS basket_description,
    f.id AS fruit_id,
    f.name AS fruit_name,
    f.fruit AS fruit_type,
    f.color AS fruit_color,
    f.weight AS fruit_weight,
    bf.create_time AS added_to_basket
FROM
    Basket b
JOIN
    BasketFruits bf ON b.basket_id = bf.basket_id
JOIN
    Fruits f ON bf.fruit_id = f.id
ORDER BY
    b.basket_id, f.id;
```

---

Rezultatas

```zsh
| Basket ID | Basket Description | Fruit ID | Fruit Name       | Fruit Type | Fruit Color | Fruit Weight | Timestamp               |
|-----------|--------------------|----------|------------------|------------|-------------|--------------|-------------------------|
| 1         | Basket 1           | 1        | Granny Smith     | Apple      | Green       | 150.5        | 2024-09-04 19:25:24.967 |
| 1         | Basket 1           | 1        | Granny Smith     | Apple      | Green       | 150.5        | 2024-09-04 19:25:24.967 |
| 1         | Basket 1           | 4        | Navel            | Orange     | Orange      | 180.3        | 2024-09-04 19:25:24.967 |
| 2         | Basket 2           | 2        | Red Delicious    | Apple      | Red         | 160.0        | 2024-09-04 19:25:24.967 |
| 2         | Basket 2           | 5        | Valencia         | Orange     | Orange      | 190.1        | 2024-09-04 19:25:24.967 |
| 3         | Basket 3           | 3        | Golden Delicious | Apple      | Yellow      | 170.2        | 2024-09-04 19:25:24.967 |
| 3         | Basket 3           | 6        | Blood Orange     | Orange     | Red         | 200.4        | 2024-09-04 19:25:24.967 |
```

---

### Ką dar galima padaryti su lokalia Vertica?

Be tai, kad nepriklausytume nuo centrinės duomenų bazės ir turėtume viską lokaliai, nes galime parsisiųsti duomenų kopiją pas save, ir kad nenaudotume interneto kelių norėdami patestuoti ar mūsų duomenų sukėlimas veikia sėkmingai, taip pat galime išsibandyti skirtingų Vertica versijų veikimą bei savo apsirašytų funkcijų (angl. _User Defined Function_) veikimą. Žinai kam dar galima panaudoti lokalę Vertica duomenų bazę? Parašyk man!

### Kaip suprasti pradinės duomenų bazės veikimo mechanizmą?

Vertica CE (Community Edition) Docker atvaizdą (Dockerfile) galima rasti Github [platformoje](https://github.com/vertica/vertica-containers/blob/main/one-node-ce/Dockerfile_Ubuntu). Ten galime rasti šio Docker atvaizdo pirminę [komandą](https://github.com/vertica/vertica-containers/blob/f5ee09d6f25cc89f092db05fa319649e1531d658/one-node-ce/Dockerfile_Ubuntu#L230) (ENTRYPOINT) bei prieš tai einančias eilutes, aprašančias jos kintamuosius.

```zsh
ENV ENTRYPOINT_SCRIPT="docker-entrypoint.sh"
ENV ENTRYPOINT_SCRIPT_PATH="${VERTICA_HOME_DIR}/${ENTRYPOINT_SCRIPT}"
ENTRYPOINT $ENTRYPOINT_SCRIPT_PATH
```

Taigi, Vertica Docker konteineris visuomet paleis "docker-entrypoint.sh" rašmeną. Gerai, tuomet toliau žiūrime kas sėdi šios rašmenos viduj.

Randame šioj rašmenoj kodą, kuris būtent ir prasuka visas rašmenas, esančias inicializacijos [direktorijoje](https://github.com/vertica/vertica-containers/blob/f5ee09d6f25cc89f092db05fa319649e1531d658/one-node-ce/Dockerfile_Ubuntu#L230).

```zsh
if [ -d /docker-entrypoint-initdb.d/ ]; then
    echo "Running entrypoint scripts ..."
    for f in $(ls /docker-entrypoint-initdb.d/* | sort); do
        case "$f" in
            *.sh)     echo "$0: running $f"; . "$f" ;;
            *.sql)    echo "$0: running $f"; ${VSQL} -f $f; echo ;;
            *)        echo "$0: ignoring $f" ;;
        esac
        echo
    done
fi
```

#### Kaip veikia pradinės duomenų bazės VMart sukūrimo mechanizmas?

Įdomumo dėlei norėčiau pažiūrėti kaip Vertica konteineris sukuria VMart duomenų bazę ir lenteles. Taigi, randu toje pačioje "docker-entrypoint.sh" rašmenoj, kad anksčiau ji prasuka kitą [rašmeną](https://github.com/vertica/vertica-containers/blob/f5ee09d6f25cc89f092db05fa319649e1531d658/one-node-ce/docker-entrypoint.sh#L139), kuri yra aprašyta po VMART_ETL_SCRIPT kintamuoju.

```zsh
echo 'Starting up'
if [ ! -d ${VERTICA_DATA_DIR}/${VERTICA_DB_NAME} ]; then
    # first time through --- create db, etc.
    mkdir -p ${VERTICA_DATA_DIR}/config
    preserve_config
    echo 'Creating database'

    ${ADMINTOOLS} -t create_db \
                  --skip-fs-checks \
                  -s localhost \
                  --database=$VERTICA_DB_NAME \
                  --catalog_path=${VERTICA_DATA_DIR} \
                  --data_path=${VERTICA_DATA_DIR}

    echo
    echo 'Loading VMart schema ...'
    ${VMART_DIR}/${VMART_ETL_SCRIPT}

    if [ -n "${APP_DB_USER}" ]; then
        create_app_db_user
    fi
else
    # ${VERTICA_OPT_DIR}/config/admintools.conf is the unmodified container
    # copy, but we symlinked it the first time through, and have to
    # recreate that symlink
    preserve_config
    echo 'Starting Database'
    ${ADMINTOOLS} -t start_db \
                  --database=$VERTICA_DB_NAME \
                  --noprompts
fi
```

VMART_ETL_SCRIPT kintamąjį randu aprašytą Vertica atvaizdo [kode](https://github.com/vertica/vertica-containers/blob/f5ee09d6f25cc89f092db05fa319649e1531d658/one-node-ce/Dockerfile_Ubuntu#L144).

```zsh
ENV VMART_ETL_SCRIPT="01_load_vmart_schema.sh"
```

Dar prieš tai randu [direktoriją](https://github.com/vertica/vertica-containers/blob/f5ee09d6f25cc89f092db05fa319649e1531d658/one-node-ce/Dockerfile_Ubuntu#L141), iš kurios paleidžiama ši rašmena


```zsh
ENV VMART_DIR="${VERTICA_OPT_DIR}/examples/VMart_Schema"
```

O dar vėliau randu, kad rašmenos iš lokalios "vmart" directorijos [perkeliamos](https://github.com/vertica/vertica-containers/blob/f5ee09d6f25cc89f092db05fa319649e1531d658/one-node-ce/Dockerfile_Ubuntu#L218) į Docker atvaizdo VMART_DIR direktoriją.

```zsh
ADD ./vmart/${VMART_ETL_SQL} ./vmart/${VMART_ETL_SCRIPT} ${VMART_DIR}/
```

Dabar belieka surasti "01_load_vmart_schema.sh" rašmeną. Ją randu Github "vmart" direktorijoje. Štai čia ji [aprašyta](https://github.com/vertica/vertica-containers/blob/main/one-node-ce/vmart/01_load_vmart_schema.sh). 

```zsh
#!/usr/bin/env bash

# (c) Copyright [2021-2023] Open Text.
# Licensed under the Apache License, Version 2.0 (the "License");
# You may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

VSQL="${VERTICA_OPT_DIR}/bin/vsql -U ${VERTICA_DB_USER}"

LOAD_CHECK_STRING="ALREADY_LOADED"
LOAD_CHECK_QUERY="select case when count(*) > 0 then '${LOAD_CHECK_STRING}' end from tables
  where table_schema = '${VMART_CONFIRM_LOAD_SCHEMA}' and table_name = '${VMART_CONFIRM_LOAD_TABLE}'"
ALREADY_LOADED=$($VSQL -c "${LOAD_CHECK_QUERY}" | grep ${LOAD_CHECK_STRING} | sed 's/ //g' || true)

if [ "${ALREADY_LOADED}" != "${LOAD_CHECK_STRING}" ]; then

  echo "Dropping old schema ..."
  cd ${VMART_DIR} && $VSQL -f vmart_schema_drop.sql

  VMART_END_YEAR=$((1 + $(date +%Y)))
  # VMART_START_YEAR=$((${VMART_END_YEAR} - ${VMART_YEARS:-4}))
  # unfortunately, all the queries in our tests are in the early part
  # of the century
  VMART_START_YEAR=2003

  echo "Generating data ..."
  cd ${VMART_DIR} && ./vmart_gen \
    --datadirectory ./ \
    --store_sales_fact 5000000 \
    --product_dimension 500 \
    --store_dimension 50 \
    --promotion_dimension 100 \
    --years "${VMART_START_YEAR}-${VMART_END_YEAR}" \
    --time_file Time_custom.txt

  echo "Creating schema ..."
  cd ${VMART_DIR} && $VSQL -f vmart_define_schema.sql

  echo "Loading files ..."
  cd ${VMART_DIR} && $VSQL -f vmart_load_data.sql

  echo "Running ETL ..."
  cd ${VMART_DIR} && $VSQL -f ${VMART_ETL_SQL}

  echo "Confirm successful load"
  $VSQL -c "create table if not exists ${VMART_CONFIRM_LOAD_SCHEMA}.${VMART_CONFIRM_LOAD_TABLE} (a int)"
else
  echo "Nothing to load, ALREADY_LOADED=${ALREADY_LOADED}"
fi
```

Nustembu matydamas, kad ši rašmena naudoja labai panašią logiką nustatyti ar duomenų bazė yra sukurta ir jei ji sukurta, tuomet rašmena nieko papildomo nedaro.

Vietoj to, kad patikrintų ar egzistuoja lentelė, ši rašmena tikrina ar ši lentelė turi bent vieną eilutę.

```sql
SELECT
  CASE WHEN COUNT(*) > 0 THEN '${LOAD_CHECK_STRING}' END
FROM tables
WHERE
  table_schema = '${VMART_CONFIRM_LOAD_SCHEMA}'
  AND table_name = '${VMART_CONFIRM_LOAD_TABLE}
```

Palyginkime su mūsų panaudota SQL užklausa.

```sql
SELECT EXISTS (
  SELECT 1 FROM v_catalog.tables WHERE table_name = 'InitiateDatabase'
);
```

Užklausos minimaliai skiriasi, tačiau yra panašios ir jų idėja yra ta pati.

### Apibendrinimas

Tai ir viskas! Dabar žinome kaip 

- Pasileisti Vertica duomenų bazę lokaliai ant Docker
- Automatizuoti pradinės duomenų bazės sukūrimą
- Sukūrėme pavyzdinę normalizuotą duomenų bazę ir apžvelgėme SQL užklausas, su kuriomis galime susirinkti aktualią informaciją iš jos
- Apžvelgėme galimus lokalios Vertica duomenų bazės panaudojimo būdus
- Išsiaiškinome pradinės duomenų bazės veikimo mechanizmą
- Išsiaiškinome pradinės duomenų bazės VMart veikimo mechanizmą

---

Kaip įdomu dabar skaityti apie darbą iš namų. Pandemijos metu darbas iš ofiso nejučia persikėlė į namų virtuves bei miegamuosius susiliedamas su mūsų kasdienybe. Atsiriboti nuo darbo po darbo laiko daugeliui pasirodė sudėtinga, žmonės dalinosi įvairiais metodais tai supaprastinti. Dabar jis jau žingsnis po žingsnio, tačiau griežtai sugrįžo į ofisus net ir programuotojų pasauly, nors ir visas darbas vyksta kompiutery. Tiesa, dabar jis hibridinis, paprastai tris dienas per savaitę. Kitur sako, kad galima dirbti nuotoliu, bet įmonė "vertina" ėjimą į ofisą. Suprantu tuomet, kad nevertina darbo iš namų, kad ir kaip šauniai dirbtum. Tuomet nuolatinis darbas iš namų yra tik nepasiekiamas marketinginis saldainis pritraukti darbuotojus. Ne visur, tačiau daug didžiųjų technologijų įmonių pasekė šiuo pavyzdžiu, kaip pasekė pavyzdžiu ir dirbti iš namų, kai pandemija prasidėjo. Nors ir daugelis teigė, kad tai daro dėl darbuotojų gerovės, panašu, kad abu judėjimai tėra mados, kuriomis seka net ir didžiosios pasaulio kompanijos.

---

_Iš milijonierių balkonų į Martą galantiškai tiesėsi rankos su gėlėmis ir taurėmis._

_--- Kokteiliuką, panele?.. Mieloji plaštake, gal akimirkai stabteltum tarp mūsų?_

_Ji pleveno ir juokėsi iš laimės (bet iš tiesų krito):_

_--- Ne, bičiuliai, ačiū. Negaliu. Skubu._

_--- Skubi? Kur? --- klausdavo jie._

_--- Ak, neklauskit, --- atsakydavo Marta ir draugiškai pamojuodavo atsisveikindama._

_Vienas jaunuolis, aukštas solidus brunetas, pabandė ją prisitraukti prie savęs. Jis jai patiko. Vis dėlto Marta iškart išsisuko:_

_--- Kaip jūs drįstate, pone?_

_Ir dar spėjo pirštu bakstelėti jam į nosies galiuką. Tie prašmatnūs žmonės ja domisi --- ši mintis glostė Martai širdį. Ji jautėsi žavinga, madinga. Žydinčiose terasose, kur zujo baltai apsirengę padavėjai ir netilo egzotiškos melodijos, žmonės kelias minutes, o gal ir trumpiau, sušnekdavo apie tą prošal skriejančią merginą (iš viršaus į apačią, vertikalus kritimas). Vieniems ji buvo graži, kitiems šiaip sau, bet visi rodė susidomėjimą._

Iš Dino Buzzati lietuviškai išleistos knygos "Šuo, kuris matė Dievą" apsakymo "Krentančioji".

___

### Šaltiniai

- Vertica Community Edition (CE) https://www.vertica.com/docs/11.0.x/HTML/Content/Authoring/GettingStartedGuide/DownloadingAndStartingVM/DownloadingAndStartingVM.htm
- docker-compose.yml https://github.com/AzisK/Vertica/blob/main/docker-compose.yml
- Docker Hub vertica/vertica-ce https://hub.docker.com/r/vertica/vertica-ce/tags
- Koks angliško kompiuterijos termino „script“ lietuviškas atitikmuo? https://vlkk.lt/konsultacijos/2699-script-rasmenys-scenarijus
- Vertica Dockerfile https://github.com/vertica/vertica-containers/blob/main/one-node-ce/Dockerfile_Ubuntu
