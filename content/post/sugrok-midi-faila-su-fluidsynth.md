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

### Kaip sužinoti, kuriuo kanalu groja MIDI failas?

MIDI failai gali naudoti skirtingus kanalus (0-15). Neradau paprasto būdo kaip išsiaiškinti kurie kanalai naudojami groti MIDI failą. Galime panaudoti `uvx` įrankį ir `mido` biblioteką. Tuomet šia  komanda ir Python rašmena galime sužinoti kokie kanalai yra naudojami:

```bash
uvx --from mido python3 -c "
import mido
mid = mido.MidiFile('tavo_failas.mid')
channels = set()
for track in mid.tracks:
    for msg in track:
        if hasattr(msg, 'channel'):
            channels.add(msg.channel)
print('Naudojami kanalai:', sorted(channels))
"
```

Mano atveju atsakymas gaunasi toks:

```bash
Naudojami kanalai: [3]
```

Tai reiškia, kad MIDI failas groja tik 3-uoju kanalu.

Dar reikia turėti omeny, kad MIDI failas gali turėti kanalo nustatymų (`program_change`) žinutes, kurios perrašo numatytuosius instrumentus. Tai galime patikrinti šia Python rašmena:

```bash
uvx --from mido python3 -c "
import mido
mid = mido.MidiFile('tavo_failas.mid')
for track in mid.tracks:
    for msg in track:
        if msg.type == 'program_change':
            print(f'Kanalas {msg.channel} -> Instrumentas {msg.program}')
"
```

Jei nieko nerodo, tai reiškia, kad failas naudoja numatytuosius instrumentus.

### Instrumentų keitimas

Pamatyti visus garso banko prieinamus instrumentus galiu taip:

```bash
inst 1
```

Tuomet matau 128 General MIDI instrumentų sąrašą:

```
000-000 Yamaha Grand Piano
000-001 Bright Yamaha Grand
000-024 Nylon String Guitar
000-040 Violin
000-056 Trumpet
...
```

Norėdamas pamatyti, kokie instrumentai groja kiekvienu kanalu:

```bash
channels -verbose
```

Pavyzdys:

```
chan 0, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 1, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 2, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 3, sfont 1, bank 0, preset 40, Yamaha Grand Piano
chan 4, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 5, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 6, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 7, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 8, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 9, sfont 1, bank 128, preset 0, Standard
chan 10, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 11, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 12, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 13, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 14, sfont 1, bank 0, preset 0, Yamaha Grand Piano
chan 15, sfont 1, bank 0, preset 0, Yamaha Grand Piano
```

Pastaba: 9-asis kanalas visada yra mušamieji (būgnai).

Norėdamas pakeisti instrumento numerį kanale, rašau šią komandą:

```bash
prog <kanalas> <instrumento_nr>
```

Pavyzdyzdžiui, keičiu 3 kanalą į elektrinį pianiną (2):

```bash
prog 3 2
```

Dar galiu naudoti select komandą:

```bash
select <kanalas> <sfont> <bank> <program>
```

Štai taip pakeičiu 3 kanalą į smuiką:

```bash
select 3 1 0 40
```

### Kaip įrašyti į WAV failą?

Jeigu gražiai skamba, tuomet įrašau į WAV failą išvesties -F parametru:

```bash
fluidsynth -ni -F rezultatas.wav ~/soundfonts/FluidR3_GM.sf2 tavo_failas.mid
```

Dabar galime klausytis bet kuriame grotuve arba konvertuoti į MP3.

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
