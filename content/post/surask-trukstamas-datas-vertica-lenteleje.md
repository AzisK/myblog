---
title: Surask trūkstamas datas Vertica lentelėje
date: 2024-10-13T04:00:00Z
author: "Ąžuolas Krušna"
tags: ["Vertica", "Duomenų bazės", "Programų inžinerija", "Duomenų inžinerija"]
---

Labas! Šiandien sužinosime kaip surasti trūkstamas datas Vertica lentelėje.

Tiesiog taip --- labai paprastai! Štai kaip čia!

```sql
WITH start_and_end AS (
    SELECT '2024-01-01'::TIMESTAMP AS tm
    UNION ALL
    SELECT '2024-12-31'::TIMESTAMP AS tm
)
SELECT DATE(ts) dates
FROM start_and_end
TIMESERIES ts AS '1 day' OVER (ORDER BY tm)
MINUS
SELECT DATE_COLUMN FROM OUR_INTERESTING_TABLE
ORDER BY dates
```

Mums tereikia pasikeisti OUR_INTERESTING_TABLE į mus dominančią lentelę ir DATE_COLUMN į jos datos stulpelį.

## Kaip tai veikia?

Gerai, pasiaiškiname kas čia vyksta. Taigi, pirmiausia mes apsirašome intervalo pradžios ir pabaigos datas. Tebūnie tai 2024 metų pradžios ir pabaigos datos:

```sql
SELECT '2024-01-01'::TIMESTAMP AS tm
UNION ALL
SELECT '2024-12-31'::TIMESTAMP AS tm
```

Rezultatas:

||tm|
|-|-|
|1|2024-01-01 00:00:00.000|
|2|2024-12-31 00:00:00.000|

Tada pasinaudojame Vertica TIMESERIES sąlyga, kad gautume visas dienas 2024 metais:

```sql
WITH start_and_end AS (
    SELECT '2024-01-01'::TIMESTAMP AS tm
    UNION ALL
    SELECT '2024-12-31'::TIMESTAMP AS tm
)
SELECT DATE(ts) dates
FROM start_and_end
TIMESERIES ts AS '1 day' OVER (ORDER BY tm)
```

Rezultatas:

||tm|
|-|-|
|1|2024-01-01|
|2|2024-01-02|
|...|...|
|365|2024-12-30|
|366|2024-12-31|

Tuomet mums belieka mus dominančios lentelės datas atimti iš šios laiko eilutės panaudojant MINUS arba EXCEPT operatorių:

```sql
WITH start_and_end AS (
    SELECT '2024-01-01'::TIMESTAMP AS tm
    UNION ALL
    SELECT '2024-12-31'::TIMESTAMP AS tm
)
SELECT DATE(ts) dates
FROM start_and_end
TIMESERIES ts AS '1 day' OVER (ORDER BY tm)
MINUS
SELECT DATE_COLUMN FROM OUR_INTERESTING_TABLE
ORDER BY dates
```

## Praktika

Taigi, susikuriame lentelę duomenų bazėje! Ši lentelė bus laiko eilutė pagal datą ir parodys kiek turėjome lankytojų tą dieną. Informacija renkama rugsėjo mėnesiui. Specialiai praleidžiame tam tikras dienas.

```sql
CREATE TABLE traffic (
    datestamp DATE,
    visitors INT
);

INSERT INTO traffic (datestamp, visitors)
SELECT DATE '2024-09-01', 10 UNION ALL
SELECT DATE '2024-09-02', 20 UNION ALL
SELECT DATE '2024-09-03', 30 UNION ALL
SELECT DATE '2024-09-05', 50 UNION ALL  -- Missing 2024-09-04
SELECT DATE '2024-09-06', 60 UNION ALL
SELECT DATE '2024-09-08', 80 UNION ALL  -- Missing 2024-09-07
SELECT DATE '2024-09-09', 90 UNION ALL
SELECT DATE '2024-09-11', 110 UNION ALL -- Missing 2024-09-10
SELECT DATE '2024-09-12', 120 UNION ALL
SELECT DATE '2024-09-14', 140 UNION ALL -- Missing 2024-09-13
SELECT DATE '2024-09-15', 150 UNION ALL
SELECT DATE '2024-09-17', 170 UNION ALL -- Missing 2024-09-16
SELECT DATE '2024-09-18', 180 UNION ALL
SELECT DATE '2024-09-20', 200 UNION ALL -- Missing 2024-09-19
SELECT DATE '2024-09-21', 210 UNION ALL
SELECT DATE '2024-09-23', 230 UNION ALL -- Missing 2024-09-22
SELECT DATE '2024-09-24', 240 UNION ALL
SELECT DATE '2024-09-26', 260 UNION ALL -- Missing 2024-09-25
SELECT DATE '2024-09-27', 270 UNION ALL
SELECT DATE '2024-09-29', 290 UNION ALL -- Missing 2024-09-28
SELECT DATE '2024-09-30', 300;
```

Nuo Vertica versijos 11.1.1 galima pasinaudoti tokia INSERT sąlyga:

```sql
INSERT INTO traffic (datestamp, visitors) VALUES
('2024-09-01', 10),
('2024-09-02', 20),
('2024-09-03', 30),
('2024-09-05', 50),  -- Missing 2024-09-04
('2024-09-06', 60),
('2024-09-08', 80),  -- Missing 2024-09-07
('2024-09-09', 90),
('2024-09-11', 110), -- Missing 2024-09-10
('2024-09-12', 120),
('2024-09-14', 140), -- Missing 2024-09-13
('2024-09-15', 150),
('2024-09-17', 170), -- Missing 2024-09-16
('2024-09-18', 180),
('2024-09-20', 200), -- Missing 2024-09-19
('2024-09-21', 210),
('2024-09-23', 230), -- Missing 2024-09-22
('2024-09-24', 240),
('2024-09-26', 260), -- Missing 2024-09-25
('2024-09-27', 270),
('2024-09-29', 290), -- Missing 2024-09-28
('2024-09-30', 300);
```

Tuomet pasinaudodami trūkstamų datų suradimo rašmena surandame visas datas, kurių trūksta šioje laiko eilutėje rugsėjo mėnesį!

```sql
WITH start_and_end AS (
    SELECT '2024-09-01'::TIMESTAMP AS tm
    UNION ALL
    SELECT '2024-09-30'::TIMESTAMP AS tm
)
SELECT DATE(ts) dates
FROM start_and_end
TIMESERIES ts AS '1 day' OVER (ORDER BY tm)
MINUS
SELECT datestamp FROM traffic
ORDER BY dates
```

Rezultatas:

||dates|
|-|-|
|1|2024-09-04|
|2|2024-09-07|
|3|2024-09-10|
|4|2024-09-13|
|5|2024-09-16|
|6|2024-09-19|
|7|2024-09-22|
|8|2024-09-25|
|9|2024-09-28|

Taigi štai taip paprastai galime rasti trūkstamas datas rugsėjo mėnesį šioje laiko eilutėje!

## O galima smulkiau?

Taip, žinoma! Naudodami Vertica TIMESERIES sąlygą galim iteruoti ne tik kas dieną, bet ir kas valandą, kas minutę, kas sekundę!

### Iteruojame kas valandą

```sql
WITH start_and_end AS (
    SELECT '2024-01-01 00:00:00'::TIMESTAMP AS tm
    UNION ALL
    SELECT '2024-01-01 04:00:00'::TIMESTAMP AS tm
)
SELECT ts
FROM start_and_end
TIMESERIES ts AS '1 hour' OVER (ORDER BY tm);
```

Rezultatas:

||ts|
|-|-|
|1|2024-01-01 00:00:00.000|
|2|2024-01-01 01:00:00.000|
|3|2024-01-01 02:00:00.000|
|4|2024-01-01 03:00:00.000|
|5|2024-01-01 04:00:00.000|

### Iteruojame kas minutę

```sql
WITH start_and_end AS (
    SELECT '2024-01-01 00:00:00'::TIMESTAMP AS tm
    UNION ALL
    SELECT '2024-01-01 00:05:00'::TIMESTAMP AS tm
)
SELECT ts
FROM start_and_end
TIMESERIES ts AS '1 minute' OVER (ORDER BY tm);
```

Rezultatas:

||ts|
|-|-|
|1|2024-01-01 00:00:00.000|
|2|2024-01-01 00:01:00.000|
|3|2024-01-01 00:02:00.000|
|4|2024-01-01 00:03:00.000|
|5|2024-01-01 00:04:00.000|
|6|2024-01-01 00:05:00.000|

### Iteruojame kas sekundę

```sql
WITH start_and_end AS (
    SELECT '2024-01-01 00:00:00'::TIMESTAMP AS tm
    UNION ALL
    SELECT '2024-01-01 00:00:01'::TIMESTAMP AS tm
)
SELECT ts
FROM start_and_end
TIMESERIES ts AS '1 second' OVER (ORDER BY tm);
```

Rezultatas:

||ts|
|-|-|
|1|2024-01-01 00:00:00.000|
|2|2024-01-01 00:00:01.000|

## O stambiau?

Žinoma, galima ir stambiau! Galime iteruoti kas 2, kas 7 dienas, kas mėnesį, kas metus.

### Iteruojame kas 2 dienas

```sql
WITH start_and_end AS (
    SELECT '2024-01-01'::TIMESTAMP AS tm
    UNION ALL
    SELECT '2024-01-04'::TIMESTAMP AS tm
)
SELECT DATE(ts) datestamp
FROM start_and_end
TIMESERIES ts AS '2 day' OVER (ORDER BY tm);
```

Rezultatas:

||datestamp|
|-|-|
|1|2024-01-01|
|2|2024-01-03|

### Iteruojame kas 7 dienas

```sql
WITH start_and_end AS (
    SELECT '2024-01-01'::TIMESTAMP AS tm
    UNION ALL
    SELECT '2024-01-31'::TIMESTAMP AS tm
)
SELECT DATE(ts) datestamp
FROM start_and_end
TIMESERIES ts AS '7 day' OVER (ORDER BY tm);
```

Rezultatas:

||datestamp|
|-|-|
|1|2023-12-30|
|2|2024-01-06|
|3|2024-01-13|
|4|2024-01-20|
|5|2024-01-27|

Matome, kad rezultatas prasideda ne nuo 2024-01-01, kas yra pirmadienis, o nuo 2023-12-30. Taip yra todėl, kad Vertica 7 dienas iteruoja kaip kas savaitę ir pradeda nuo šeštadienio, o jeigu pradžia yra ne šeštadienis, tai pradeda nuo praeito šeštadienio. Šiuo atveju 2023-12-30. Visuomet pravartu patikrinti kaip Vertica interpretuoja intervalą ir per jį iteruoja.

---

_Kelioms sekundėms stojo tyla, paskui iš maišalienės Arturo smegenyse išsisunkė keletas žodžių._

_--- Triša Makmilan? --- išspaudė jis. --- Ką čia veiki?_

_--- Tą patį, ką ir tu, --- atsakė ji. --- Mane paėmė pavežti. O kas dar man liko daryti, turint matematikos ir astrofizikos mokslų diplomus? Turėjau rinktis: arba tai, arba pirmadienį vėl į eilę prie bedarbio pašalpų._ 

Iš Douglas Adams lietuviškai išleistos knygos "Keliautojo autostopu gidas po galaktiką".

---

## Šaltiniai

- TIMESERIES clause https://docs.vertica.com/24.2.x/en/sql-reference/statements/select/timeseries-clause/
- MINUS clause https://docs.vertica.com/23.3.x/en/sql-reference/statements/select/minus-clause/
- New & changed in Vertica 11.1.1 / SQL functions & statements https://docs.vertica.com/11.1.x/en/new-features/11.1.1/sql-functions-and-statements/
- Gap filling and interpolation (GFI) https://docs.vertica.com/24.2.x/en/data-analysis/time-series-analytics/gap-filling-and-interpolation-gfi/
