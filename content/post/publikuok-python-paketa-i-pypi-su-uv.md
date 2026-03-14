---
title: Publikuok Python paketą į PyPI su uv
description: Kaip sukurti, sukompiliuoti ir publikuoti Python paketą į PyPI naudojant uv
date: 2026-03-09T06:00:00Z
author: Ąžuolas Krušna
tags: ["Programų inžinerija", "Python", "uv", "PyPI"]
ShowCodeCopyButtons: true
---

Labas! Šiandien sužinosime kaip nuo nulio susikurti ir publikuoti Python paketą į PyPI su uv programa. uv yra labai greita, todėl programavimas su ja dar smagesnis!

### Įsidiegiame uv

Jeigu dar neturi uv, įsidiek jį terminale:

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Įsitikink, kad veikia:

```bash
uv --version
```

### Sukuriame naują biblioteką

uv turi patogią komandą projektui inicijuoti. Naudosime `--lib` vėliavėlę, kuri sukuria biblioteką (angl. *library*) --- paketą, skirtą naudoti kituose projektuose.

```bash
uv init --lib mano-paketas
cd mano-paketas
```

Ši komanda sukurs tokią projekto struktūrą:

```
mano-paketas/
├── .python-version
├── README.md
├── pyproject.toml
└── src/
    └── mano_paketas/
        ├── __init__.py
        └── py.typed
```

Kuriant paketą, uv naudoja `src` struktūrą (angl. _src layout_). Tai reiškia, kad paketo kodas gyvena `src/` direktorijoje, o ne projekto šaknyje.

Kodėl tai svarbu? Be `src/` struktūros, Python importuotų tavo modulį tiesiogiai iš projekto šaknies --- testuotum šaltinio failus, o ne tai, ką gaus naudotojas. Su `src/` struktūra, `uv run` pirmiau instaliuoja paketą į laikiną aplinką, tada importuoja --- taip testuoji **tikrą instaliuotą paketą**, kaip ir jį gautų kiti žmonės. Tai padeda pastebėti paketo konfigūracijos klaidas (pamiršti failai, neteisingi path'ai).

### Peržvelgiame pyproject.toml

Sugeneruotas `pyproject.toml` failas atrodys maždaug taip:

```toml
[project]
name = "mano-paketas"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.11"
dependencies = []

[build-system]
requires = ["uv_build>=0.10.2,<0.11.0"]
build-backend = "uv_build"
```

Čia mums svarbiausia:

- `[project]` --- paketo metaduomenys: pavadinimas, versija, aprašymas, priklausomybės.
- `[build-system]` --- kompiliavimo sistema. uv naudoja savo `uv_build` pagal nutylėjimą, bet galima naudoti ir `hatchling`, `flit-core`, `setuptools` ar kitas.

Papildome metaduomenis pagal savo poreikius: aprašymą, autorių, licenciją:

```toml
[project]
name = "mano-paketas"
version = "0.1.0"
description = "Mano nuostabus Python paketas"
readme = "README.md"
requires-python = ">=3.11"
license = "MIT"
authors = [
    { name = "Mano Vardas", email = "mano@email.com" },
]
dependencies = []
```

### Rašome kodą

Atsidarome `src/my_package/__init__.py` ir aprašome savo paketo logiką. Pavyzdžiui:

```python
def hello(vardas: str = "Pasauli") -> str:
    return f"Labas, {vardas}!"
```

Galime iškart patikrinti veikimą:

```bash
uv run python -c "import mano_paketas; print(mano_paketas.hello('PyPI'))"
```

Turėtum pamatyti:

```
Labas, PyPI!
```

### Pridedame priklausomybes (jei reikia)

Jeigu tavo paketas naudoja kitas bibliotekas, pridėk jas su `uv add`:

```bash
uv add requests
```

Tai automatiškai atnaujins `pyproject.toml` bei `uv.lock` failą.

### Kompiliuojame paketą

Kai kodas paruoštas, sukompiliuojame jį į distribucijos formatus (angl. *source distribution* ir *wheel*):

```bash
uv build
```

Rezultatas atsiras `dist/` direktorijoje:

```
dist/
├── mano_paketas-0.1.0-py3-none-any.whl
└── mano_paketas-0.1.0.tar.gz
```

- `.tar.gz` --- tai šaltinio (angl. _source_) distribucija su visu kodu.
- `.whl` --- tai sukompiliuota dvejetainė (angl. _wheel_) distribucija, kuri įsidiegia greičiau.

### PyPI paskyra ir API raktas

Prieš publikuojant reikia:

1. Susikurti paskyrą [pypi.org](https://pypi.org/account/register/).
2. Susigeneruoti API raktą (angl. *API token*): eik į [Account Settings → API tokens](https://pypi.org/manage/account/#api-tokens) ir sukurk naują raktą.

PyPI nepalaiko slaptažodžio autentifikacijos, todėl būtinas API raktas.

Rekomenduoju pirmiausia išbandyti su [TestPyPI](https://test.pypi.org/) --- tai PyPI kopija testavimui, kur galima bandyti publikavimą be pasekmių. Susikurk paskyrą ir API raktą ten atskirai.

### Publikuojame į TestPyPI

Pirmausia, apsirašome TestPyPI indeksą savo `pyproject.toml`:

```toml
[[tool.uv.index]]
name = "testpypi"
url = "https://test.pypi.org/simple/"
publish-url = "https://test.pypi.org/legacy/"
explicit = true
```

Tuomet publikuojame:

```bash
uv publish --index testpypi --token tavo-testpypi-api-raktas
```

Tikrinam ar paketas pasiekiamas:

```bash
uv run --with mano-paketas --index-url https://test.pypi.org/simple/ --no-project -- python -c "import mano_paketas; print(mano_paketas.hello())"
```

### Publikuojame į PyPI

Kai viskas veikia su TestPyPI, publikuojame į tikrąjį PyPI:

```bash
uv publish --token tavo-pypi-api-raktas
```

Dabar jau tavo paketas pasiekiamas visiems!

Tikrinam ar paketas teisingai įsidiegia:

```bash
uv run --with mano-paketas --no-project -- python -c "import mano_paketas; print(mano_paketas.hello())"
```

`--no-project` vėliavėlė užtikrina, kad paketas bus instaliuotas iš PyPI, o ne iš lokalios projekto direktorijos.

### Atnaujiname versiją

Norėdami išleisti naują paketo versiją, galime naudoti uv `version` subkomandą:

```bash
# Nustatyti konkrečią versiją
uv version 1.0.0

# Pakelti minor versiją (0.1.0 → 0.2.0)
uv version --bump minor

# Pakelti patch versiją (0.1.0 → 0.1.1)
uv version --bump patch
```

Pakėlus paketo versiją, ją išleisti galime ir vėl šiomis komandomis: `uv build` ir `uv publish`.

### Jei nepavyko

Jei kažkas neveikia:

1. Ar paketo pavadinimas laisvas? Tikrinam `https://pypi.org/project/tavo-paketas/`.
2. Ar API raktas teisingas ir prasideda `pypi-`? TestPyPI ir PyPI raktai yra skirtingi!
3. Ar versija nauja? PyPI neleidžia perrašyti jau publikuotos versijos --- pakelk su `uv version --bump patch`.
4. Ar `uv build` pavyksta? Patikrink, kad `pyproject.toml` turi teisingą `[build-system]` skiltį ir kad `src/` struktūra atitinka paketo pavadinimą.

---

Štai ir viskas! Laukiu išbandyti tavąjį paketą!

---

> Moksleivė
>
> Su naujienomis, su naujienomis\
> Aš namo jau sugrįžau:\
> Aš -- moksleivė, gimnazistė --\
> Viską, viską jau žinau:
>
> Kad ta žemė ne kaip blynas,\
> Kad nesveika gerti vynas,\
> Kad dukart po du ne trys,\
> Kad diena -- tai ne naktis...
>
> Kad aukštai kitokis oras,\
> Kad lakštingala ne voras,\
> Kad šalelė prigimta\
> Tai yr mūsų Lietuva...
>
> Su naujienomis, su naujienomis\
> Aš namo jau sugrįžau:\
> Aš -- moksleivė, gimnazistė --\
> Viską, viską jau žinau!

1933 m. parašytas Matildos Olkinaitės eilėraštis.

---

### Šaltiniai

- uv dokumentacija: Building and publishing a package https://docs.astral.sh/uv/guides/package/
- uv dokumentacija: Working on projects https://docs.astral.sh/uv/guides/projects/
- uv dokumentacija: Creating projects https://docs.astral.sh/uv/concepts/projects/init/
- PyPI: Python Package Index https://pypi.org/
- uv GitHub repozitorija https://github.com/astral-sh/uv
