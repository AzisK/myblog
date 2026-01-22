---
title: Susikurk Redis klasterį lokaliai
description: Kaip susikurti Redis klasterį savo kompiuteryje naudojant paprastus bash skriptus
date: 2024-04-22T04:00:00Z
author: "Ąžuolas Krušna"
tags: ["Programų inžinerija", "Redis", "DevOps"]
draft: true
ShowCodeCopyButtons: true
---

Programuotojau, kviečiu Tave susikurti Redis klasterį savo kompiuteryje! Kam to reikia? Na, galbūt nori išbandyti, kaip Tavo aplikacija elgsis su tikru Redis klasteriu, o galbūt tiesiog nori geriau suprasti, kaip ši technologija veikia. Bet kuriuo atveju --- einam!

### Kas yra Redis klasteris?

Redis klasteris yra Redis duomenų bazės paskirstyta versija, kuri leidžia duomenis padalinti tarp kelių mazgų (nodes). Tai suteikia didesnį patikimumą ir galimybę apdoroti daugiau užklausų vienu metu.

**Kaip veikia duomenų paskirstymas?**

Redis klasteris naudoja **16 384 hash slot'us**. Kiekvienas raktas (key) priskiriamas vienam iš šių slot'ų pagal formulę `CRC16(key) mod 16384`. Kiekvienas pagrindinis mazgas (master) yra atsakingas už dalį šių slot'ų.

Kai klientas nori pasiekti raktą, jis kreipiasi į bet kurį mazgą. Jei raktas priklauso kitam mazgui, Redis grąžina `MOVED` atsakymą su teisingu adresu --- ir klientas automatiškai persijungia.

Minimaliam klasteriui reikia **6 mazgų**: 3 pagrindinių (master) ir 3 replikų (slave). Replikos užtikrina, kad duomenys nebus prarasti, jei vienas iš pagrindinių mazgų sugenda.

### Ko reikės?

Prieš pradedant, įsitikink, kad turi įdiegtą [Redis](https://redis.io/docs/getting-started/installation/). Jei naudoji macOS:

```bash
brew install redis
```

**Svarbu apie portus:** kiekvienas Redis klasterio mazgas naudoja **du TCP portus**:
1. Kliento portas (pvz., 7000) --- įprastoms Redis komandoms
2. Klasterio magistralės portas (kliento portas + 10000, pvz., 17000) --- mazgų tarpusavio komunikacijai

Įsitikink, kad abu portai yra laisvi kiekvienam mazgui!

### Greičiausias būdas: create-cluster skriptas

Redis ateina su įrankiu, kuris viską padaro už tave. Jį rasi Redis šaltinio kode `utils/create-cluster` direktorijoje, arba jei įsidiegei per Homebrew:

```bash
cd /opt/homebrew/opt/redis/share/redis
```

Tada tiesiog:

```bash
./create-cluster start   # Paleidžia 6 Redis mazgus
./create-cluster create  # Sukuria klasterį (atsakyk "yes")
```

Klasteris veikia ant portų 30001-30006. Prisijunk su:

```bash
redis-cli -c -p 30001
```

Kai baigsi:

```bash
./create-cluster stop    # Sustabdo mazgus
./create-cluster clean   # Išvalo duomenis
```

### Rankinis būdas: minimalus pavyzdys

Jei nori daugiau kontrolės arba neturi `create-cluster` įrankio, štai paprastas būdas.

**1. Sukurk direktorijas kiekvienam mazgui:**

```bash
mkdir -p ~/redis-cluster/{7000,7001,7002,7003,7004,7005}
```

**2. Sukurk minimalų `redis.conf` kiekvienoje direktorijoje:**

```bash
for port in 7000 7001 7002 7003 7004 7005; do
cat > ~/redis-cluster/$port/redis.conf << EOF
port $port
cluster-enabled yes
cluster-config-file nodes-$port.conf
cluster-node-timeout 5000
dir ~/redis-cluster/$port
EOF
done
```

Tai sukurs 6 konfigūracijos failus su minimaliomis nuostatomis:
- `port` --- portas, kuriuo klausysis mazgas
- `cluster-enabled yes` --- įjungia klasterio režimą
- `cluster-config-file` --- failas, kuriame Redis saugos klasterio būseną
- `cluster-node-timeout` --- laikas milisekundėmis, po kurio mazgas laikomas nepasiekiamu
- `dir` --- direktorija duomenims

**3. Paleisk visus mazgus:**

```bash
for port in 7000 7001 7002 7003 7004 7005; do
    redis-server ~/redis-cluster/$port/redis.conf &
done
```

**4. Sukurk klasterį:**

```bash
redis-cli --cluster create \
    127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 \
    127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 \
    --cluster-replicas 1
```

Parametras `--cluster-replicas 1` reiškia, kad kiekvienas master turės po vieną repliką. Redis paprašys patvirtinti konfigūraciją --- atsakyk `yes`.

**5. Sustabdyti klasterį:**

```bash
for port in 7000 7001 7002 7003 7004 7005; do
    redis-cli -p $port SHUTDOWN
done
```

### Patikrinimas

Prisijunk prie klasterio naudodamas `-c` vėliavėlę (klasterio režimas):

```bash
redis-cli -c -p 7000 cluster info
```

Turėtum matyti:

```
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_known_nodes:6
cluster_size:3
```

Ką reiškia šie laukai:
- `cluster_state:ok` --- klasteris veikia ir gali priimti užklausas
- `cluster_slots_assigned:16384` --- visi slot'ai priskirti mazgams
- `cluster_known_nodes:6` --- klasteris žino apie 6 mazgus
- `cluster_size:3` --- 3 pagrindiniai mazgai aptarnauja slot'us

Pamatyti visus mazgus:

```bash
redis-cli -c -p 7000 cluster nodes
```

### Išbandyk klasterį

Pabandyk įrašyti duomenis:

```bash
$ redis-cli -c -p 7000
127.0.0.1:7000> set foo bar
-> Redirected to slot [12182] located at 127.0.0.1:7002
OK
127.0.0.1:7002> get foo
"bar"
```

Matai `Redirected` žinutę? Redis automatiškai nukreipė tave į mazgą, kuris atsakingas už rakto `foo` slot'ą (12182).

### Hash tag'ai: kai reikia kelių raktų tame pačiame mazge

Kartais reikia atlikti operacijas su keliais raktais vienu metu (pvz., `MGET`, `MSET`). Redis klasteryje tai veikia tik jei visi raktai yra tame pačiame slot'e.

Tam naudok **hash tag'us** --- `{...}` sintaksę:

```bash
127.0.0.1:7000> set user:{123}:name "Jonas"
127.0.0.1:7000> set user:{123}:email "jonas@example.com"
127.0.0.1:7000> mget user:{123}:name user:{123}:email
1) "Jonas"
2) "jonas@example.com"
```

Visi raktai su `{123}` pateks į tą patį slot'ą, nes tik dalis tarp `{` ir `}` naudojama hash skaičiavimui.

### Jei nepavyko

Jei kažkas neveikia:
1. Ar Redis įdiegtas? `redis-server --version`
2. Ar portai laisvi? `lsof -i :7000`
3. Ar magistralės portai laisvi? `lsof -i :17000`

Pradėti iš naujo:

```bash
rm -rf ~/redis-cluster/*/nodes-*.conf
rm -rf ~/redis-cluster/*/dump.rdb
```

### Ką dar verta žinoti

**Apie Docker:** Redis klasteris neveikia su NAT ir portų peradresavimu. Docker konteineriams naudok `--net=host` režimą:

```bash
# Paleisk 6 Redis mazgus Docker konteineriuose
for port in 7000 7001 7002 7003 7004 7005; do
    docker run -d --name redis-$port --net=host redis:latest \
        redis-server --port $port --cluster-enabled yes \
        --cluster-config-file nodes-$port.conf --cluster-node-timeout 5000
done

# Sukurk klasterį
docker run --rm --net=host redis:latest \
    redis-cli --cluster create \
    127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 \
    127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 \
    --cluster-replicas 1

# Sustabdyti ir išvalyti
docker stop redis-{7000,7001,7002,7003,7004,7005}
docker rm redis-{7000,7001,7002,7003,7004,7005}
```

**Pastaba:** `--net=host` veikia tik Linux sistemose. macOS ir Windows vartotojams yra keletas variantų:

**Greičiausias būdas --- viena eilutė:**

```bash
docker run -d -p 7000-7005:7000-7005 -e IP=0.0.0.0 grokzen/redis-cluster:latest
```

**Arba su Bitnami:**

```yaml
# docker-compose.yml
services:
  redis-cluster:
    image: bitnami/redis-cluster:latest
    environment:
      - REDIS_CLUSTER_REPLICAS=1
      - REDIS_CLUSTER_CREATOR=yes
      - ALLOW_EMPTY_PASSWORD=yes
    ports:
      - "6379:6379"
```

**Jei nori suprasti, kas vyksta "po gaubtu"** --- štai išsamus `docker-compose.yml` pavyzdys:

```yaml
services:
  redis-node-1:
    image: redis:latest
    command: redis-server --port 6379 --cluster-enabled yes --cluster-announce-ip 172.20.0.11
    networks:
      redis-cluster:
        ipv4_address: 172.20.0.11

  redis-node-2:
    image: redis:latest
    command: redis-server --port 6379 --cluster-enabled yes --cluster-announce-ip 172.20.0.12
    networks:
      redis-cluster:
        ipv4_address: 172.20.0.12

  redis-node-3:
    image: redis:latest
    command: redis-server --port 6379 --cluster-enabled yes --cluster-announce-ip 172.20.0.13
    networks:
      redis-cluster:
        ipv4_address: 172.20.0.13

  redis-node-4:
    image: redis:latest
    command: redis-server --port 6379 --cluster-enabled yes --cluster-announce-ip 172.20.0.14
    networks:
      redis-cluster:
        ipv4_address: 172.20.0.14

  redis-node-5:
    image: redis:latest
    command: redis-server --port 6379 --cluster-enabled yes --cluster-announce-ip 172.20.0.15
    networks:
      redis-cluster:
        ipv4_address: 172.20.0.15

  redis-node-6:
    image: redis:latest
    command: redis-server --port 6379 --cluster-enabled yes --cluster-announce-ip 172.20.0.16
    networks:
      redis-cluster:
        ipv4_address: 172.20.0.16

networks:
  redis-cluster:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

Paleidus `docker-compose up -d`, sukurk klasterį:

```bash
docker exec -it redis-node-1 redis-cli --cluster create \
    172.20.0.11:6379 172.20.0.12:6379 172.20.0.13:6379 \
    172.20.0.14:6379 172.20.0.15:6379 172.20.0.16:6379 \
    --cluster-replicas 1
```

Esmė --- `--cluster-announce-ip` nurodo, kokį IP adresą mazgas skelbia kitiems. Be šios nuostatos mazgai matytų Docker vidinius adresus ir negalėtų susisiekti.

**Apie duomenų saugumą:** Redis klasteris naudoja asinchroninę replikaciją --- pagrindinis mazgas patvirtina įrašymą prieš replikuojant duomenis. Yra mažas langas, kuriame patvirtinti įrašymai gali būti prarasti, jei mazgas sugenda prieš replikaciją.

**Apie klientų bibliotekas:** aplikacijose naudok cluster-aware klientus:

| Kalba | Biblioteka | Nuoroda |
|-------|------------|---------|
| Python | redis-py | [github.com/redis/redis-py](https://github.com/redis/redis-py) |
| Node.js | ioredis | [github.com/redis/ioredis](https://github.com/redis/ioredis) |
| Java | Jedis | [github.com/redis/jedis](https://github.com/redis/jedis) |
| Go | go-redis | [github.com/redis/go-redis](https://github.com/redis/go-redis) |
| C# | StackExchange.Redis | [github.com/StackExchange/StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis) |
| Ruby | redis-rb | [github.com/redis/redis-rb](https://github.com/redis/redis-rb) |

***

Redis klasteris lokaliai yra puikus būdas išmokti, kaip veikia paskirstytos sistemos. Daugiau informacijos rasite oficialioje [Redis Cluster dokumentacijoje](https://redis.io/docs/latest/operate/oss_and_stack/management/scaling/).

_May the force be with you_
