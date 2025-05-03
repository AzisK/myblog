---
title: Draugiškos Python klasės
date: 2024-10-29T07:00:00Z
ShowCodeCopyButtons: true
---

Labas, šiandien noriu pasidalinti draugiškomis Python klasėmis. Tai tokios klasės, su kuriomis man dirbti smagiausia.

## Instrumentai, sukurti draugiškai klasei

Nulipdyti draugišką klasę naudosimės 
- Duomenų klasėmis (dataclasses), 
- Operatorių aprašymu (operator overloading)
- Kompozicijos vietoj paveldėjimo principu (composition over inheritance)

## Taigi, kokios tos draugiškos Python klasės? 

A. Jas lengva suprasti ir su jomis susikalbėti (lengva skaityti ir rašyti)\
B. Suprantame vienas kitą be žodžių (operatoriais)\
C. Jos yra lengvai pasiekiamos (kompozicija vietoj paveldimumo)

Pažvelkime kaip praktiškai galėtų arodyti šios savybės.

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

### B. Suprantame vienas kitą be žodžių (operatoriais)

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

### C. Lengvai pasiekiamos (kompozicija vietoj paveldimumo)

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

Grąžina

```python
MetricV10(alias='Revenue_/_Searches_x_1000', field='((SUM(revenue)) / (SUM(searches)) * 1000)')
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

NL = '\n'

@dataclass
class Cube:
  metrics: list[MetricV10]
  dimensions: list[Dimension]
  table: str

  def __post_init__(self):
    self.create_time = datetime.now()

  @property
  def sql(self):
    return f"""
SELECT
  {f',{NL}  '.join(f"{d.field} AS '{d.alias}'" for d in self.dimensions)},
  {f',{NL}  '.join(f"{m.field} AS '{m.alias}'" for m in self.metrics)}
FROM {self.table}
GROUP BY {', '.join(d.field for d in self.dimensions)}
-- Query created: {self.create_time:%Y-%m-%d %H:%M:%S}
    """.strip()
```

Dabar sukuriame mūsų kubą, pavadinkime jį Minervos kubu (CubeMinerva). Minerva buvo romėnų išminties deivė ir buvo žinoma dėl jos pasiekimų medicinoje, prekyboje, poezijoje bei menuose. 

Iš dimensijų kubui pridedame tik mėnesio dimensiją, kitos dimensijos mūsų apibendrintai analizei nėra aktualios.

```python
metrics = [MetricRevenue, MetricSessions, MetricSearches, MetricRevenuePerSessions, MetricRpm, MetricSessionsBounced, MetricSessionsBouncedPercent]
dimensions = [DimensionMonth]
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
  SUM(revenue) AS 'Revenue',
  SUM(sessions) AS 'Sessions',
  SUM(searches) AS 'Searches',
  (SUM(revenue)) / (SUM(sessions)) AS 'Revenue_/_Sessions',
  ((SUM(revenue)) / (SUM(searches)) * 1000) AS 'Revenue_/_Searches_x_1000',
  SUM(CASE WHEN views=1 THEN sessions ELSE 0 END) AS 'SessionsBounced',
  ((SUM(CASE WHEN views=1 THEN sessions ELSE 0 END)) / (SUM(sessions)) * 100) AS 'SessionsBounced_/_Sessions_x_100'
FROM Minerva
GROUP BY MONTH(ts)
-- Query created: 2024-10-28 07:47:16
```

## Apibendrinimas

Štai ir viskas. Apžvelgėme instrumentus, kuriuos panaudojome lipdydami draugiškas Python klases --- duomenų klasės, operatoriai ir kompozicija.

Tuomet apsibrėžėme draugiškas Python klases. Prisimename:

A. Jas lengva suprasti ir su jomis susikalbėti (lengva skaityti ir rašyti)\
B. Suprantame vienas kitą be žodžių (operatoriais)\
C. Jos yra lengvai pasiekiamos (kompozicija vietoj paveldimumo)

Draugiškų Python klasių pagalba sukūrėme metrikų klases bei objektus. 

Praktikos daly išplėtėme metrikos galimybes papildydami dalybos operatorių bei aprašydami daugybos operatorių.

Papildėme mūsų kodą dimensijos klase bei objektais, analitinio kubo klase ir objektu, grąžinančiu analitinį SQL kodą.

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