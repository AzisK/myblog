---
title: Draugiškos Python klasės
date: 2024-10-17T12:00:00Z
ShowCodeCopyButtons: true
draft: true
---

Labas, šiandien noriu pasidalinti draugiškomis Python klasėmis. Tai tokios klasės, su kuriomis per savo darbo metus man dirbti smagiausia.

Taigi, kokios tos draugiškos Python klasės? 

- Jas lengva suprasti ir su jomis susikalbėti (lengva skaityti ir rašyti)
- Jų nesunku paprašyti paslaugos (aprašyti logiką)
- Vienas kitą galime suprantame iš gestų (operatorių)
- Jos yra atviros (kompozicija vietoj paveldimumo)

### A. Lengva suprasti ir susikalbėti (skaityti ir rašyti)

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

### B. Nesunku paprašyti paslaugos (aprašyti logiką)

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

### C. Suprasti vienas kitą iš gestų (operatorių)

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

Norėdami suskaičiuoti kai pajamų uždirbame per vieną paiešką, metriką galime apsirašyti taip

```python
MetricSearches = MetricV7(alias='Searches', field='SUM(searches)')

MetricRevenue / MetricSearches
```

Grąžins

```python
MetricV7(alias='Revenue_/_Searches', field='(SUM(revenue)) / (SUM(searches))')
```

### D. Atviros (kompozicija vietoj paveldimumo)

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

Gerai, dabar naudodamiesi draugiškomis klasėmis sukursime analitinį kubą!

### Praktika. Analitinis kubas

Pirmiausia, išplečiame metrikos klasės operatorių galimybes taip, kad galėtume metrikas dalinti ne tik iš klasių, bet ir iš skaičių, o ir skaičius dalinti iš metrikų. Kartu sukuriame galimybę metrikas dauginti iš skaičių bei galimybę metrikas dauginti iš skaičių ir skaičius dauginti iš metrikų.

Tai galime pasiekti 

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

Kubas!


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
GROUP BY {','.join(d.alias for d in self.dimensions)}
-- Cube created: {self.create_time:%Y-%m-%d %H:%M:%S}
    """.strip()
```

___



___

### Šaltiniai

- Overloading arithmetic operators with dunder methods https://mathspp.com/blog/pydonts/overloading-arithmetic-operators-with-dunder-methods
- Data Classes in Python 3.7+ (Guide) https://realpython.com/python-data-classes/
- The Composition Over Inheritance Principle https://python-patterns.guide/gang-of-four/composition-over-inheritance/


### Tolimesniam skaitymui

- Python Data Classes: A Comprehensive Tutorial https://www.datacamp.com/tutorial/python-data-classes
- How to Use Python Data Classes in 2023 (A Beginner’s Guide) https://www.dataquest.io/blog/how-to-use-python-data-classes/
- Data Classes in Kotlin https://kotlinlang.org/docs/data-classes.html