---
title: Sugrok MIDI failą su FluidSynth
description: Sugrok MIDI failą su FluidSynth terminale
date: 2026-03-08T08:00:00Z
author: Ąžuolas Krušna
tags: ["MIDI", "Muzika", "Terminalas"]
ShowCodeCopyButtons: true
---

Labas, šiandien grosime MIDI failus terminale!

Aš nutariau pažvelgti į savo magistro darbą šviežiomis ausimis ir akimis. Jau po daugelio metų -- daugiau nei septynių! Jau kaip patyręs programuotojas. Tuo metu kūriau muziką su rekursiniais neuroniniais tinklais -- aido neuroniniais tinklais. Džiaugiuosi tuometine savo ambicija ir drąsa. Tačiau šiandien ne apie tai. Šiandien apie MIDI failų sugrojimą!

Tam mes naudojame FluidSynth programą.

### Kas yra FluidSynth?

FluidSynth yra programinis sintezatorius (angl. *software synthesizer*), kuris skaito MIDI failus ir paverčia juos garsu naudodamas SoundFont (`.sf2`) garso bankus. SoundFont --- tai failas, kuriame saugomi tikrų instrumentų garso pavyzdžiai (angl. *samples*). Kai FluidSynth gauna MIDI instrukciją, pavyzdžiui, "pagrok man do natą fortepijonu", jis paima atitinkamą įrašytą do natos skambesį iš Soundfont failo ir mums sugroja.

### Kaip įsidiegti FluidSynth?

Kadangi aš naudoju macOS operacinę sistemą, tad įsidiegiu FluidSynth naudojant Homebrew programą. Kaip įsidiegti kitose operacinėse sistemose, žiūrėkite [FluidSynth čia](https://github.com/FluidSynth/fluidsynth/wiki/Download).

```bash
brew install fluid-synth
```

Tikrinu ar sėkmingai

```bash
fluidsynth --version
```

Turėtume matyti kažką panašaus į tai

```bash
FluidSynth runtime version 2.5.3
```

### Kaip parsisiųsti SoundFont garso banką?

Dabar mums trūksta tik SoundFont garso banko failo. Be jo FluidSynth neturės ką sugroti, nes jis tik groja, o pačių skambesių įrašų neturi.

Populiariausias nemokamas SoundFont yra FluidR3 GM, kuriame yra 128 standartiniai instrumentai: fortepijonas, gitara, smuikas, trimitai ir kiti.

Žemiau galime rasti rašmeną, kur parsiunčia mums SoundFont failą

```bash
mkdir -p ~/soundfonts  # Sukuria soundfonts aplanką vartotojo namų direktorijoje
curl -L -o ~/soundfonts/FluidR3_GM.zip "https://keymusician01.s3.amazonaws.com/FluidR3_GM.zip"  # Parsisiunčia archyvuotą SoundFont failą
unzip ~/soundfonts/FluidR3_GM.zip -d ~/soundfonts # Išarchyvuoja SoundFont failą iš archyvo
```

Tikrinam ar failas teisingas

```bash
file ~/soundfonts/FluidR3_GM.sf2
```

Matome, kad failas yra SoundFont/Bank formatas, todėl viskas gerai

```bash
FluidR3_GM.sf2: RIFF (little-endian) data, SoundFont/Bank
```

### Grojimas

Štai ir viskas, ką reikėjo paruošti! Dabar paleisk savo MIDI failą:

```bash
fluidsynth -a coreaudio -ni ~/soundfonts/FluidR3_GM.sf2 tavo_failas.mid
```

Ką reiškia šie Fluidsynth parametrai?

- `-a coreaudio` --- naudoti macOS garso sistemą (angl. _audio driver_)
- `-ni` --- neinteraktyvus režimas (angl. _non-interactive_), t.y. programa gros failą ir baigs darbą
- pirmas failas --- SoundFont garso bankas
- antras failas --- tavo MIDI failas

Jeigu tiek ir reikėjo, tuomet sveikinu! Gali uždaryti šį puslapį ir mėgautis muzika. O jei nori sužinoti daugiau, skaityk toliau.

### Papildomi parametrai

#### Garso stiprumas

Jei per tyliai arba per garsiai, naudok `-g` parametrą:

```bash
fluidsynth -a coreaudio -ni -g 2.0 ~/soundfonts/FluidR3_GM.sf2 tavo_failas.mid
```

Reikšmė 1.0 yra numatytoji, 2.0 yra dvigubai garsiau, o 0.5 yra dvigubai tyliau.

#### Interaktyvus režimas

Jei nori valdyti grojimą rankiniu būdu, paleisk be `-ni`:

```bash
fluidsynth -a coreaudio ~/soundfonts/FluidR3_GM.sf2 tavo_failas.mid
```

Tada galėsi naudoti FluidSynth konsolę. Pagrindinės komandos:

| Komanda | Aprašymas |
|---------|-----------|
| `help` | Parodyti visas komandas |
| `player_start` | Pradėti grojimą nuo pradžių |
| `player_cont` | Tęsti grojimą |
| `player_stop` | Rodyti dabartinę poziciją (ne sustabdyti!) |
| `player_seek num` | Peršokti `num` tikų pirmyn/atgal |
| `gain value` | Keisti garsumą (0–5, pvz., `gain 2.0`) |
| `set synth.reverb.active False` | Išjungti aidą (reverb) |
| `set synth.chorus.active False` | Išjungti chorus efektą |
| `settings` | Rodyti visus nustatymus |
| `quit` | Išeiti |

Patarimas: įvesk `help` norėdamas pamatyti visas komandas. Garsumo nustatymas: `gain 2.0` (garsiau nei numatytoji reikšmė). Numatytoji reikšmė priklauso nuo FluidSynth versijos: `1.0` (senesnės) arba `0.2` (naujesnės 2.x+).

#### Įrašymas į WAV failą 

Nori konvertuoti MIDI į garso failą?

```bash
fluidsynth -ni -F rezultatas.wav ~/soundfonts/FluidR3_GM.sf2 tavo_failas.mid
```

Parametras `-F` nurodo išvesties failą. Taip gausi WAV formatą, kurį galėsi klausytis bet kuriame grotuve arba konvertuoti į MP3.

### Jeigu nepavyko

Jeigu kažkas nesigauna ir gauname tokius rezultatus

1. `command not found: fluidsynth`
  
Reiškia FluidSynth neįdiegtas. Įsitikrink, kad `fluidsynth --version` grąžina versiją, o ne klaidą. Pabandyk įdiegti Fluidsynth dar kartą `brew install fluid-synth`.

2. `expected RIFF chunk id` 

SoundFont failas sugadintas arba tai ne SF2 failas. Tikrinam su `file ~/soundfonts/FluidR3_GM.sf2`

3. Negirdėti garso 

Patikrink garso stiprumą sistemoje ir bandyk pagarsintu su `-g 2.0`.

---

Štai ir viskas, dabar galėsime groti MIDI failus iš terminalo iki kol kaimynai užpyks. Smagaus _rokenrolo_!

---

MIDI formato pradžia siekia 1983-uosius metus, kai Dave Smith ir Ikutaro Kakehashi sukūrė standartą, leidžiantį skirtingiems muzikos instrumentams „kalbėtis" tarpusavyje. Už šį darbą jie abu gavo techninį Grammy apdovanojimą 2013-aisiais. Įdomu, kad standartas beveik nepasikeitė per 40 metų --- MIDI failai, sukurti praėjusio amžiaus devintajame dešimtmetyje, puikiai veikia ir šiandien.

---

### Šaltiniai

- FluidSynth oficiali svetainė https://www.fluidsynth.org/
- FluidR3 GM SoundFont garso bankas https://member.keymusician.com/Member/FluidR3_GM/
- FluidSynth dokumentacija https://github.com/FluidSynth/fluidsynth/wiki
