---
title: Draugiškos Python klasės
date: 2024-10-17T12:00:00Z
ShowCodeCopyButtons: true
# draft: true
---

Labas, šiandien noriu pasidalinti draugiškomis Python klasėmis. Tai tokios klasės, su kuriomis per savo darbo metus man dirbti smagiausia.

Taigi, kokios tos draugiškos Python klasės? 

A. Jas lengva suprasti ir su jomis susikalbėti (lengva skaityti ir rašyti)\
B. Jų nesunku paprašyti paslaugos (aprašyti logiką)\
C. Vienas kitą galime suprantame iš gestų (operatorių)\
D. Jos yra atviros (kompozicija vietoj paveldimumo)

## A. Lengva suprasti ir susikalbėti (skaityti ir rašyti)

Pažvelkime į tradicinį būdą perduoti kintamuosius klasei inicijavimo metu naudojant `__init__` metodą:
```python
class MetricV1:
  def __init__(self, alias, field):
    self.alias = alias
    self.field = field
```

Toks objektas atspausdins savo modulio pavadinimą, klasės pavadinimą ir adresą atminty. Visai nieko informacija tiriant kompiuterinę programą, tačiau iš dalies perteklinė, paini ir mažai ką pasakanti informacija apie patį objektą.
```python
MetricV1(alias='Revenue', field='SUM(revenue)')
```

Grąžina
```python
<__main__.MetricV1 at 0x7eba60ae8fd0>
```

Manau, kad daug draugiškesnė klasės objekto reprezentacija programų inžinieriui yra tokia:
```python
MetricV2(alias='Revenue', field='SUM(revenue)')
```

Grąžina
```python
MetricV2(alias='Revenue', field='SUM(revenue)')
```

Taip mes matome visus argumentus, kurie buvo paduoti klasei, pagal kurią buvo sukurtas objektas. Ši objekto reprezentacija yra tokia pat kaip ir aprašymas kuriant šį objektą.

Tai mes galime pasiekti apsirašę klasės *dunder* metodą `__repr__`:

```python
class MetricV2:
  def __init__(self, alias, field):
    self.alias = alias
    self.field = field

  def __repr__(self):
    return f"{self.__class__.__name__}(alias='{self.alias}', field='{self.field}')"
```

Taip mano klasės objektai tampa daug draugiškesni skaityti ir suprasti, tačiau viską galime supaprastinti pasinaudodami Python duomenų klasėmis `dataclass`!

Viskas, ko mums prireiks, tai importuoti duomenų klasių modulį bei panaudoti duomenų klasės dekoratorių virš klasės! Tuomet mums tereikia aprašyti galimus klasės kintamuosius sufleruojant (angl. *type-hint*) jų duomenų tipą (angl. *data-type*). Tik tiek, ir turime visą anksčiau turėtą funkcionalumą ir net daugiau.

Duomenų klasės yra Python standartinės bibliotekos nuo 3.7 Python versijos.

```python
from dataclasses import dataclass

@dataclass
class MetricV3:
  alias: str
  field: str
```

Gerai, o ar šios klasės lanksčios? Taip, labai lanksčios! Jos kintamieji gali turėti ir reikšmes pagal nutylėjimą!

```python
from dataclasses import dataclass

@dataclass
class MetricV4:
  alias: str
  field: str
  translation: str = 'Metrika'
```

Šio objekto
```python
MetricV4(alias='Revenue', field='SUM(revenue)')
```

rašytinė reprezentacija bus tokia:
```python
MetricV4(alias='Revenue', field='SUM(revenue)', translation='Metrika')
```

O šio objekto
```python
MetricV4(alias='Revenue', field='SUM(revenue)', translation='Pajamos')
```

rašytinė reprezentacija bus tokia:
```python
MetricV4(alias='Revenue', field='SUM(revenue)', translation='Pajamos')
```

Kartu duomenų klasės gali turėti ir kintamuosius, kurie nėra perduodami klasei objekto inicijavimo metu. Tokiu atveju nereikia sufleruoti kintamojo duomenų tipo:

```python
from dataclasses import dataclass

@dataclass
class MetricV5:
  alias: str
  field: str
  product = 'Duomenų Analizė'
```

Šio objekto 
```python
MetricV5(alias='Revenue', field='SUM(revenue)')
```

rašytinė reprezentacija bus tokia:
```python
MetricV5(alias='Revenue', field='SUM(revenue)')
```

O pagrąžinus jo vidinį "product" kintamąjį, 

```python
MetricV5(alias='Revenue', field='SUM(revenue)').product
```

turėsime anksčiau apsirašytą reikšmę:
```python
'Duomenų Analizė'
```

## B. Nesunku paprašyti paslaugos (aprašyti logiką)

O kas, jeigu mums vis dėlto inicijavimo metu reikia papildomos logikos, kuriai galimybę suteikia `__init__` metodas?

Viskas gerai, tokios galimybės mes neprarandame su duomenų klasėmis. Tam galime pasinaudoti `__post_init__` metodu, kuriuo galime apsirašyti visą norimą logiką jau perdavus klasės objektui parametrus.

Pavyzdžiui, `__post_init__` metode aprašysime objekto kintamąjį "datetime_create" ir perduosime jam objekto sukūrimo laiką.

```python
from datetime import datetime
from dataclasses import dataclass

@dataclass
class MetricV6:
  alias: str
  field: str

  def __post_init__(self):
      self.datetime_create = datetime.now()
```

Matome, kad objekto sukūrimo laiko mes neperdavėme inicijuodami objektą, tačiau jis buvo pridėtas ir yra pasiekiamas:

```python
MetricRevenue = MetricV6(alias='Revenue', field='SUM(revenue)')

MetricRevenue.datetime_create
```

Atspaudina

```python
datetime.datetime(2024, 10, 17, 12, 10, 52, 123456)
```

## C. Suprasti vienas kitą iš gestų (operatorių)

Nepaprastai džiugu, kai draugas supranta ką galvoju iš kūno kalbos. Tik truputį mažiau smagu, kai klasės gali bendrauti operatoriais. 

Kadangi mes kaip pavyzdį pasirinkome metrikos klases, kurios turi savo "alias" bei SQL sąlygos lauką "field", tai galėtume pridėti galimybę dalinti vieną metriką iš kitos! Santykinės metrikos gali labai praversti duomenų analizei.

Tam, kad aprašytume dalybos operatorių, naudojame *dunder* metodą `__truediv__`. 

Santykinės metrikos "alias" lauką aprašome kaip vienos metrikos "alias" iš kitos metrikos "alias" tarpus atskirdami "_", o vidury įterpdami dalybos simbolį "/". Pavyzdžiui, "alias1\_/_alias2". 

Santykinės metrikos "field" lauką aprašome kaip vieno santykį iš kito ir dar apskliaudžiame. Apskliaudžiame todėl, kad jeigu darytume dar ilgesnę išvestinę metriką, tai kad išliktų ši veiksmų seka, nepaisant kokia aritmetika vyktų po ar prieš šį dalybos veiksmą.

```python
from dataclasses import dataclass

@dataclass
class MetricV7:
  alias: str
  field: str

  def __truediv__(self, other):
      return MetricV7(alias=f'{self.alias}_/_{other.alias}', field=f'({self.field}) / ({other.field})')
```

Metode `__truediv__` "self" yra klasė esanti kairėje dalybos ženklo pusėje, o "other" yra klasė, esanti dešinėje dalybos ženklo pusėje.

Tuomet

```python
MetricRevenue = MetricV7(alias='Revenue', field='SUM(revenue)')
MetricSessions = MetricV7(alias='Sessions', field='SUM(sessions)')

MetricRevenue / MetricSessions
```

Grąžins

```python
MetricV7(alias='Revenue_/_Sessions', field='(SUM(revenue)) / (SUM(sessions))')
```

Norėdami suskaičiuoti kiek vidutiniškai pajamų atneša viena paieška, metriką galime apsirašyti taip

```python
MetricSearches = MetricV7(alias='Searches', field='SUM(searches)')

MetricRevenue / MetricSearches
```

Grąžins

```python
MetricV7(alias='Revenue_/_Searches', field='(SUM(revenue)) / (SUM(searches))')
```

## D. Atviros (kompozicija vietoj paveldimumo)

Šios klasės atviros, nes jos nėra painios. Įvairių klasių ir jų subklasių sistema nesunkiai gali labai sunkiai suprantamai išsišakoti ne tik giliai, bet ir plačiai. Ši problema vadinama subklasių sprogimu (angl. *subclass explosion*). Šiai problemai išvengti galima pasinaudoti kompozicijos vietoj paveldimumo principu. Šis principas teikia pirmumą objektų kompozicijai vietoj klasių paveldimumo.

Šiuo mūsų paprastu atveju šį principą galima pateikti taip. Jeigu eitume paveldimumo keliu, tuomet turėtume bazinę metrikos klasę, kuri turėtų reikiamą logiką, o visas kitas metrikas aprašytume kaip klases, paveldinčias šią bazinę mmetriką.

Tuomet turėtume

```python
class BaseMetric:
  alias: str
  field: str

  def __repr__(self):
    return f"{self.__class__.__name__}(alias='{self.alias}', field='{self.field}')"
```

Taigi, mes čia nenaudojame nei duomenų klasių, nei perduodame kintamuosius *`__init__`* metodui inicijuojant objektus.

Aprašyti kiekvienos metrikos "alias" ir "field" kintamuosius paveldėsime šią klasę ir aprašysime jos kintamuosius klasės aprašyme. Tai atrodys taip:

```python
class RevenueMetric(BaseMetric):
  alias = 'Revenue'
  field = 'SUM(revenue)'

class SessionsMetric(BaseMetric):
  alias = 'Sessions'
  field = 'SUM(sessions)'

class SearchesMetric(BaseMetric):
  alias = 'Searches'
  field = 'SUM(searches)'
```

Kadangi aprašėme jų bazinės metrikos klasės `BaseMetric` `__repr__` metodą, kaip ir ankstesnių metrikų su šiuo metodu, tai pagrąžinus šių klasių objektus gausime tokius pačius atsakymus:

Pajamų metrika

```python
RevenueMetric()
```

Grąžina

```python
RevenueMetric(alias='Revenue', field='SUM(revenue)')
```

Sesijų metrika

```python
SessionsMetric()
```

Grąžina

```python
SessionsMetric(alias='Sessions', field='SUM(sessions)')
```

Paieškų metrika

```python
SearchesMetric()
```

Grąžina

```python
SearchesMetric(alias='Searches', field='SUM(searches)')
```

Visgi taip aprašydami objektą Python gautume klaidą: "TypeError: SearchesMetric() takes no arguments". 

Žiūrint į šį paprastą atvejį neatrodo, kad paveldėti klases yra blogas sprendimas ir kad tai prives prie klaidų. Kartu nemanau, kad šiam atvejui tai yra blogas sprendimas. Visgi matau kaip sudėtingėja kodo skaitomumas, kad kodas užima daugiau vietos, sudėtingiau interpretuoti objektus, juos perkurti. Tiesiog paveldėdami bazinę klasę taip pat negalime pasinaudoti operatoriais. Realios programos kartu turi daug daugiau logikos ir paveldimumo džiunglės tuomet auga kaip ant mielių.

---

Gerai, dabar naudodamiesi draugiškomis klasėmis sukursime minimalistinį analitinį kubą!

## E. Praktika. Analitinis kubas

### E. 1. Patogi metrika

Pirmiausia, išplečiame metrikos klasės operatorių galimybes taip, kad galėtume metrikas dalinti ne tik iš klasių, bet ir iš skaičių, o ir skaičius dalinti iš metrikų. Kartu sukuriame galimybę metrikas dauginti iš skaičių bei galimybę metrikas dauginti iš skaičių ir skaičius dauginti iš metrikų. Tai galime pasiekti palaipsniui. 

#### E. 1. 1. Dalyba iš metrikų ir skaičių

Išplečiame `MetricV7` `__true_div__` metodą taip, kad patikrintume "other" objekto duomenų tipą. Taigi, atliekame apsirašytą situaciją tik tuomet, kai "other" yra tos pačios klasės. 

Kartu analogiškai pridedame galimybę dalinti iš skaičių: "int" ir "float" duomenų tipų.

Jeigu tai kitas duomenų tipas nei skaičius ("int" ar "float") ar mūsų aprašyta metrikos klasė, tuomet grąžiname `NotImplemented`.

Dabar mūsų `MetricV8` atrodo taip:

```python
from dataclasses import dataclass

@dataclass
class MetricV8:
  alias: str
  field: str

  def __truediv__(self, other):
    if isinstance(other, (int, float)):
      return self.__class__(alias=f'{self.alias}_/_{other}', field=f'({self.field} / {other})')
    if isinstance(other, self.__class__):
      return self.__class__(alias=f'{self.alias}_/_{other.alias}', field=f'({self.field}) / ({other.field})')
    return NotImplemented
```

`NotImplemented` reikšmė yra grąžinama tuomet, kai toks funkcionalumas yra nepalaikomas.

Pavyzdžiui, dalindami `MetricV8` iš `str` duomenų tipo, gausime `TypeError` klaidą.

```python
MetricV8(alias='Revenue', field='SUM(revenue)') / 'Sessions'
```

```python
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-8-fd8d9754cda4> in <cell line: 1>()
----> 1 MetricV8(alias='Revenue', field='SUM(revenue)') / 'Sessions'

TypeError: unsupported operand type(s) for /: 'MetricV8' and 'str'
```

Dabar dalindami iš skaičių galime, pavyzdžiui, padalinti metriką iš tūkstančio, tai turėsime jos reikšmę tūkstančiais.

```python
MetricV8(alias='Revenue', field='SUM(revenue)') / 1000
```

Grąžina

```python
MetricV8(alias='Revenue_/_1000', field='(SUM(revenue) / 1000)')
```

#### E. 1. 2. Skaičių dalyba iš metrikų

Pasiekti skaičių dalybą iš metrikų, mums reikia apsirašyti *dunder* metodą `__rtruediv__`. Dalinant skaičių iš metrikos Python bando kviesti skaičiaus `__truediv__` metodą, tačiau skaičiai neturi logikos būti padalinti iš mūsų aprašytos metrikos, todėl tuomet yra kviečiamas metrikos `__rtruediv__` metodas. Jis yra kviečiamas tuomet, kai dalinamas duomenų tipas neturi `__truediv__` arba grąžina `NotImplemented`.

Jį galime aprašyti taip:

```python
from dataclasses import dataclass

@dataclass
class MetricV9:
  alias: str
  field: str

  def __truediv__(self, other):
      if isinstance(other, (int, float)):
          return self.__class__(alias=f'{self.alias}_/_{other}', field=f'({self.field} / {other})')
      if isinstance(other, self.__class__):
        return self.__class__(alias=f'{self.alias}_/_{other.alias}', field=f'({self.field}) / ({other.field})')
      return NotImplemented
  
  def __rtruediv__(self, other):
      if isinstance(other, (int, float)):
          return self.__class__(alias=f'{other}_/_{self.alias}', field=f'({other} / {self.field})')
      return NotImplemented
```

Dabar mes skaičius galime dalinti iš metrikų. Pavyzdžiui, jeigu mūsų tikslas yra per mėnesį perduoti prekių už 3 tūkstančius, tuomet galime paskaičiuoti kokią dalį to tikslo pasiekėme:

```python
3000 / MetricV9(alias='Revenue', field='SUM(revenue)')
```

Grąžina

```python
MetricV9(alias='3000_/_Revenue', field='(3000 / SUM(revenue))')
```

#### E. 1. 3. Metrikų daugyba iš skaičių ir skaičių daugyba iš metrikų. 

Analogiškai, aprašysime daugybos metodus `__mul__` ir `__rmul__`, tik `__rmul__` metodą bus aprašyti lengviau, nes dauginant dauginamųjų seka nėra svarbi, priešingai nei dalinant.

Galiausiai, mūsų metrika atrodo taip:

```python
from dataclasses import dataclass

@dataclass
class MetricV10:
  alias: str
  field: str

  def __truediv__(self, other):
      if isinstance(other, (int, float)):
          return self.__class__(alias=f'{self.alias}_/_{other}', field=f'({self.field} / {other})')
      if isinstance(other, self.__class__):
        return self.__class__(alias=f'{self.alias}_/_{other.alias}', field=f'({self.field}) / ({other.field})')
      return NotImplemented
  
  def __rtruediv__(self, other):
      if isinstance(other, (int, float)):
          return self.__class__(alias=f'{other}_/_{self.alias}', field=f'({other} / {self.field})')
      return NotImplemented

  def __mul__(self, other):
      if isinstance(other, (int, float)):
          return self.__class__(alias=f'{self.alias}_x_{other}', field=f'({self.field} * {other})')
      if isinstance(other, self.__class__):
          return self.__class__(alias=f'{self.alias}_x_{other.alias}', field=f'{self.field} * {other.field}')
      return NotImplemented

  def __rmul__(self, other):
      return self.__mul__(other)
```

#### E. 1. 4. Mums įdomios metrikos

Dabar apsirašome visas mums įdomias metrikas. Tebūnie šiam pavyzdžiui tai: pajamos, sesijos, paieškos, pajamų kiekis per sesiją ir RPM (revenue per mille), reiškiančią pajamas per tūkstantį paieškų, ir atšokusių (angl. *bounced*) sesijų kiekį bei procentą. Atšokusius sesijos, tai tokios, kuomet vartotojas atsidarė puslapį, bet nedarė nieko daugiau, paprastumo dėlei apsirašysime tokias sesijas kaip turinčias tik 1 puslapio peržiūrą.

```python
MetricRevenue = MetricV10(alias='Revenue', field='SUM(revenue)')
MetricSessions = MetricV10(alias='Sessions', field='SUM(sessions)')
MetricSearches = MetricV10(alias='Searches', field='SUM(searches)')

MetricRevenuePerSessions = MetricRevenue / MetricSessions
MetricRevenuePerSessions
```

Grąžina

```python
MetricV10(alias='Revenue_/_Sessions', field='(SUM(revenue)) / (SUM(sessions))')
```

O RPM metrika

```python
MetricRpm = MetricRevenue / MetricSearches * 1000
MetricRpm
```

Apsirašome atšokusių sesijų procentą

```python
MetricSessionsBounced = MetricV10(alias='SessionsBounced', field='SUM(CASE WHEN views=1 THEN revenue ELSE 0 END)')
MetricSessionsBouncedPercent = MetricSessionsBounced / MetricSessions * 100
MetricSessionsBouncedPercent
```

Grąžina

```python
MetricV10(alias='SessionsBounced_/_Sessions_x_100', field='((SUM(CASE WHEN views=1 THEN revenue ELSE 0 END)) / (SUM(sessions)) * 100)')
```

### E. 2. Dimensija

Apsirašysime paprastą dimensijos klasę

```python
@dataclass
class Dimension:
  alias: str
  field: str
```

Ja naudodamiesi sukursime mums įdomias dimensijas, tebūnie tai: data, svetainės domenas, puslapio kelias ir svetainė, iš kurios atėjo vartotojas ("referrer").

```python
DimensionMonth = Dimension(alias='Month', field='MONTH(ts)')
DimensionDomain = Dimension(alias='Domain', field='domain')
DimensionPath = Dimension(alias='Path', field='path')
DimensionReferrer = Dimension(alias='Referrer', field='referrer')

DimensionDate, DimensionDomain, DimensionPath, DimensionReferrer
```

Grąžina

```python
(Dimension(alias='Date', field='ts'),
 Dimension(alias='Domain', field='domain'),
 Dimension(alias='Path', field='path'),
 Dimension(alias='Referrer', field='referrer'))
```

### E. 3. Kubas

Gerai! Dabar beliko apsirašyti mūsų minimalistinį analitinį kubą, kuris grąžins mums SQL kodą, grąžinantį iš SQL duomenų bazės norimas metrikas ir dimensijas. Formuojant norimą kubo SQL užklausos kodą paprašysime iš lentelės (table) dimensijų, metrikų bei pagrupuosime pagal metrikas.

Būsimiems tyrimams ir analizei dar prie užklausos pridėsim kubo sukūrimo laiką kaip SQL komentarą per `create_time` objekto kintamąjį, pridedamą `__post_init__` metodo metu.

Taigi, mūsų kubo klasė atrodys taip:

```python
from datetime import datetime
from dataclasses import dataclass

@dataclass
class Cube:
  metrics: list[MetricV7]
  dimensions: list[Dimension]
  table: str

  def __post_init__(self):
    self.create_time = datetime.now()

  @property
  def sql(self):
    return f"""
SELECT 
  {', '.join(f'{d.field} AS {d.alias}' for d in self.dimensions)}, 
  {', '.join(f'{m.field} AS {m.alias}' for m in self.metrics)}
FROM {self.table}
GROUP BY {', '.join(d.alias for d in self.dimensions)}
-- Cube created: {self.create_time:%Y-%m-%d %H:%M:%S}
    """.strip()
```

Dabar sukuriame mūsų kubą, pavadinkime jį Minervos kubu (CubeMinerva). Minerva buvo romėnų išminties deivė ir buvo žinoma dėl jos pasiekimų medicinoje, prekyboje, poezijoje bei menuose. 

Iš dimensijų kubui pridedame tik mėnesio dimensiją, kitos dimensijos mūsų apibendrintai analizei nėra aktualios.

```python
metrics = [MetricRevenue, MetricSessions, MetricSearches, MetricRevenuePerSessions, MetricRpm, MetricSessionsBounced, MetricSessionsBouncedPercent]
dimensions = [DimensionMonth, DimensionDomain]
CubeMinerva = Cube(metrics=metrics, dimensions=dimensions, table='Minerva')
print(CubeMinerva)
```

Tuomet šio kubo "sql" kintamasis

```python
print(CubeMinerva.sql)
```

Atspaudina mums tokį SQL kodą

```sql
SELECT
  MONTH(ts) AS 'Month',
  SUM(revenue) AS 'Revenue', SUM(sessions) AS 'Sessions', SUM(searches) AS 'Searches', (SUM(revenue)) / (SUM(sessions)) AS 'Revenue_/_Sessions', ((SUM(revenue)) / (SUM(searches)) * 1000) AS 'Revenue_/_Searches_x_1000', SUM(CASE WHEN views=1 THEN sessions ELSE 0 END) AS 'SessionsBounced', ((SUM(CASE WHEN views=1 THEN sessions ELSE 0 END)) / (SUM(sessions)) * 100) AS 'SessionsBounced_/_Sessions_x_100'
FROM Minerva
GROUP BY MONTH(ts)
-- Query created: 2024-10-28 17:19:47
```
### E. 4. Analitika

#### E. 4. 1. Duombazės lentelė

Susikuriame pavyzdinę duombazės lentelę, iš kurios galėtume pasigrąžinti duomenis pagal šią užklausą. Žinoma, realus pasaulis veikia atvirkščiai --- viskas prasideda nuo duomenų ir duomenų bazių lentelių ir tik paskui yra lipdomi analitiniai kubai, tačiau kadangi mano tikslas yra pasidalinti draugiškomis Python klasėmis, tai nuo jų ir pradėjom, o duomenų lentelę sugeneruojam tik tada, kai prireikia. Taigi, štai mūsų lentelė:

```sql
CREATE TABLE Minerva (
    ts TIMESTAMP,
    domain VARCHAR(255),
    path VARCHAR(255),
    referrer VARCHAR(255),
    revenue DECIMAL(10, 2),
    sessions INT,
    searches INT,
    views INT
);
```

#### E. 4. 2. Įkeliame duomenis į lentelę

Įkeliame į ją pavyzdinius duomenis pagal mūsų apsirašytus stulpelius.

{{< collapse "Pavyzdiniai duomenys">}}
```sql
INSERT INTO Minerva (ts, domain, path, referrer, revenue, sessions, searches, views) VALUES
('2023-01-01', 'aziogas.lt', '/home', 'google.com', 100.00, 50, 200, 60),
('2023-01-01', 'aziogas.lt', '/about', 'bing.com', 120.00, 55, 220, 70),
('2023-01-01', 'aziogas.lt', '/contact', 'yahoo.com', 150.00, 60, 250, 80),
('2023-01-01', 'aziogas.lt', '/home', 'facebook.com', 0.00, 1, 10, 1),
('2023-01-01', 'aziogas.lt', '/about', 'twitter.com', 0.00, 1, 15, 1),
('2023-02-01', 'aziogas.lt', '/home', 'facebook.com', 180.00, 65, 270, 90),
('2023-02-01', 'aziogas.lt', '/about', 'twitter.com', 200.00, 70, 300, 100),
('2023-02-01', 'aziogas.lt', '/contact', 'linkedin.com', 220.00, 75, 320, 110),
('2023-02-01', 'aziogas.lt', '/home', 'direct', 0.00, 1, 20, 1),
('2023-02-01', 'aziogas.lt', '/about', 'instagram.com', 0.00, 1, 25, 1),
('2023-03-01', 'aziogas.lt', '/home', 'direct', 250.00, 80, 350, 120),
('2023-03-01', 'aziogas.lt', '/about', 'instagram.com', 280.00, 85, 370, 130),
('2023-03-01', 'aziogas.lt', '/contact', 'pinterest.com', 300.00, 90, 400, 140),
('2023-03-01', 'aziogas.lt', '/home', 'google.com', 0.00, 1, 30, 1),
('2023-03-01', 'aziogas.lt', '/about', 'bing.com', 0.00, 1, 35, 1),
('2023-04-01', 'aziogas.lt', '/home', 'google.com', 320.00, 95, 420, 150),
('2023-04-01', 'aziogas.lt', '/about', 'bing.com', 350.00, 100, 450, 160),
('2023-04-01', 'aziogas.lt', '/contact', 'yahoo.com', 380.00, 105, 470, 170),
('2023-04-01', 'aziogas.lt', '/home', 'facebook.com', 0.00, 1, 40, 1),
('2023-04-01', 'aziogas.lt', '/about', 'twitter.com', 0.00, 1, 45, 1),
('2023-05-01', 'aziogas.lt', '/home', 'facebook.com', 400.00, 110, 500, 180),
('2023-05-01', 'aziogas.lt', '/about', 'twitter.com', 420.00, 115, 520, 190),
('2023-05-01', 'aziogas.lt', '/contact', 'linkedin.com', 450.00, 120, 550, 200),
('2023-05-01', 'aziogas.lt', '/home', 'direct', 0.00, 1, 50, 1),
('2023-05-01', 'aziogas.lt', '/about', 'instagram.com', 0.00, 1, 55, 1),
('2023-06-01', 'aziogas.lt', '/home', 'direct', 480.00, 125, 570, 210),
('2023-06-01', 'aziogas.lt', '/about', 'instagram.com', 500.00, 130, 600, 220),
('2023-06-01', 'aziogas.lt', '/contact', 'pinterest.com', 520.00, 135, 620, 230),
('2023-06-01', 'aziogas.lt', '/home', 'google.com', 0.00, 1, 60, 1),
('2023-06-01', 'aziogas.lt', '/about', 'bing.com', 0.00, 1, 65, 1),
('2023-07-01', 'aziogas.lt', '/home', 'google.com', 550.00, 140, 650, 240),
('2023-07-01', 'aziogas.lt', '/about', 'bing.com', 580.00, 145, 670, 250),
('2023-07-01', 'aziogas.lt', '/contact', 'yahoo.com', 600.00, 150, 700, 260),
('2023-07-01', 'aziogas.lt', '/home', 'facebook.com', 0.00, 1, 70, 1),
('2023-07-01', 'aziogas.lt', '/about', 'twitter.com', 0.00, 1, 75, 1),
('2023-08-01', 'aziogas.lt', '/home', 'facebook.com', 620.00, 155, 720, 270),
('2023-08-01', 'aziogas.lt', '/about', 'twitter.com', 650.00, 160, 750, 280),
('2023-08-01', 'aziogas.lt', '/contact', 'linkedin.com', 680.00, 165, 770, 290),
('2023-08-01', 'aziogas.lt', '/home', 'direct', 0.00, 1, 80, 1),
('2023-08-01', 'aziogas.lt', '/about', 'instagram.com', 0.00, 1, 85, 1),
('2023-09-01', 'aziogas.lt', '/home', 'direct', 700.00, 170, 800, 300),
('2023-09-01', 'aziogas.lt', '/about', 'instagram.com', 720.00, 175, 820, 310),
('2023-09-01', 'aziogas.lt', '/contact', 'pinterest.com', 740.00, 180, 840, 320),
('2023-09-01', 'aziogas.lt', '/home', 'google.com', 0.00, 1, 90, 1),
('2023-09-01', 'aziogas.lt', '/about', 'bing.com', 0.00, 1, 95, 1),
('2023-10-01', 'aziogas.lt', '/home', 'google.com', 760.00, 185, 860, 330),
('2023-10-01', 'aziogas.lt', '/about', 'bing.com', 780.00, 190, 880, 340),
('2023-10-01', 'aziogas.lt', '/contact', 'yahoo.com', 800.00, 195, 900, 350),
('2023-10-01', 'aziogas.lt', '/home', 'facebook.com', 0.00, 1, 100, 1),
('2023-10-01', 'aziogas.lt', '/about', 'twitter.com', 0.00, 1, 105, 1),
('2023-11-01', 'aziogas.lt', '/home', 'facebook.com', 820.00, 200, 920, 360),
('2023-11-01', 'aziogas.lt', '/about', 'twitter.com', 840.00, 205, 940, 370),
('2023-11-01', 'aziogas.lt', '/contact', 'linkedin.com', 860.00, 210, 960, 380),
('2023-11-01', 'aziogas.lt', '/home', 'direct', 0.00, 1, 110, 1),
('2023-11-01', 'aziogas.lt', '/about', 'instagram.com', 0.00, 1, 115, 1),
('2023-12-01', 'aziogas.lt', '/home', 'direct', 880.00, 215, 980, 390),
('2023-12-01', 'aziogas.lt', '/about', 'instagram.com', 900.00, 220, 1000, 400),
('2023-12-01', 'aziogas.lt', '/contact', 'pinterest.com', 920.00, 225, 1020, 410),
('2023-12-01', 'aziogas.lt', '/home', 'google.com', 0.00, 1, 120, 1),
('2023-12-01', 'aziogas.lt', '/about', 'bing.com', 0.00, 1, 125, 1);
```
{{< /collapse >}}

#### E. 4. 3. Analitinė užklausa

Mūsų anksčiau suformuota analitinio kubo SQL užklausa grąžina tokius analitinius duomenis (jie nėra tikri, vartotojai nėra sekami ir puslapis negeneruoja jokių pajamų).

| Month | Revenue | Sessions | Searches | Revenue_/_Sessions | Revenue_/_Searches_x_1000 | SessionsBounced | SessionsBounced_/_Sessions_x_100 |
|-------|---------|----------|----------|--------------------|---------------------------|-----------------|----------------------------------|
| 1     | 370.00  | 167      | 695      | 2.215569           | 532.374100                | 2               | 1.1976                           |
| 2     | 600.00  | 212      | 935      | 2.830189           | 641.711229                | 2               | 0.9434                           |
| 3     | 830.00  | 257      | 1185     | 3.229572           | 700.421940                | 2               | 0.7782                           |
| 4     | 1050.00 | 302      | 1425     | 3.476821           | 736.842105                | 2               | 0.6623                           |
| 5     | 1270.00 | 347      | 1675     | 3.659942           | 758.208955                | 2               | 0.5764                           |
| 6     | 1500.00 | 392      | 1915     | 3.826531           | 783.289817                | 2               | 0.5102                           |
| 7     | 1730.00 | 437      | 2165     | 3.958810           | 799.076212                | 2               | 0.4577                           |
| 8     | 1950.00 | 482      | 2405     | 4.045643           | 810.810810                | 2               | 0.4149                           |
| 9     | 2160.00 | 527      | 2645     | 4.098672           | 816.635160                | 2               | 0.3795                           |
| 10    | 2340.00 | 572      | 2845     | 4.090909           | 822.495606                | 2               | 0.3497                           |
| 11    | 2520.00 | 617      | 3045     | 4.084279           | 827.586206                | 2               | 0.3241                           |
| 12    | 2700.00 | 662      | 3245     | 4.078550           | 832.049306                | 2               | 0.3021                           |

## Apibendrinimas

Štai ir viskas. Apsibrėžėme draugiškas Python klases. Prisimename kaip jas apsirašėm:

A. Jas lengva suprasti ir su jomis susikalbėti (lengva skaityti ir rašyti)\
B. Jų nesunku paprašyti paslaugos (aprašyti logiką)\
C. Vienas kitą galime suprantame iš gestų (operatorių)\
D. Jos yra atviros (kompozicija vietoj paveldimumo)

Apsirašėme šias savybes daugiausia (A, B, dalinai D) naudodamiesi duomenų klasėmis, bet taip pat ir apsirašydami jų tarpusavio bendravimą operatoriais (B). Kartu apsirašėme ketvirtą savybę (D) pritaikydami kompozicijos vietoj paveldimumo principą.

Draugiškų Python klasių pagalba sukūrėme metrikų klases bei objektus. Praktikos daly papildėme mūsų kodą dimensijos klase bei objektais, analitinio kubo klase ir objektu, grąžinančiu analitinį SQL kodą.

Tuomet susikūrėme pavyzdinę duombazės lentelę su duomenimis ir pritaikėme mūsų analitinio kubo SQL kodą, grąžinantį analitinius duomenis

---

Ši idėja buvo labai smarkiai įkvėpta brazilo Luciano Ramalho knygos "Fluent Python". Prie šio akiračio prisidėjo ir kompizicijos vietoj paveldimumo principo aprašymas pagal Brandon Rhodes.

___

_Tačiau dar svarbiau milžiniška rankšluosčio psichologinė vertė. Kažkodėl, jei kaušas (kaušas --- tai ne autostopininkas) pamato, kad autostopininkas nešasi rankšluostį, automatiškai padaro išvadą, jog šis taip pat turi dantų šepetėlį, nosinę, muilą, sausainių pakelį, gertuvę, kompasą, žvaigždėlapį, špagato ritinėlį, purškalą nuo uodų, lietpaltį ir skėtį, skafandrą ir taip toliau, ir panašiai. Maža to, kaušas autostopininkui visada mielai paskolins bet kurį iš minėtųjų ar tuziną kitų daiktų, kuriuos autostopininkas neva netyčia "pametė". Žvelgiant kaušo akimis, bet kas, gebantis skersai išilgai išmaišyti galaktiką, išsiversti be patogumų, glaustis lindynėse, kautis su baisiais netikėtumais, įveikti sunkumus ir vis dar žinoti, kur yra jo rankšluostis, --- tai, be abejonių, žmogus, su kuriuo reikia skaitytis._

Taip rašoma Douglas Adams romano "Keliautojo autostopu gidas po galaktiką" plotmėje egzistuojančiame tikrąjame gide "Keliautojo autostopu gidas po galaktiką".

___

### Šaltiniai

- Luciano Ramalho (2022). _Fluent Python: Clear, Concise, and Effective Programming_ (2nd ed.). O'Reilly. https://www.oreilly.com/library/view/fluent-python-2nd/9781492056348/
- Overloading arithmetic operators with dunder methods https://mathspp.com/blog/pydonts/overloading-arithmetic-operators-with-dunder-methods
- Data Classes in Python 3.7+ (Guide) https://realpython.com/python-data-classes/
- The Composition Over Inheritance Principle https://python-patterns.guide/gang-of-four/composition-over-inheritance/
- Python \_\_truediv__() Magic Method
 https://blog.finxter.com/python-__truediv__-magic-method/ 

### Tolimesniam skaitymui

- Python Data Classes: A Comprehensive Tutorial https://www.datacamp.com/tutorial/python-data-classes
- How to Use Python Data Classes in 2023 (A Beginner’s Guide) https://www.dataquest.io/blog/how-to-use-python-data-classes/
- Data Classes in Kotlin https://kotlinlang.org/docs/data-classes.html