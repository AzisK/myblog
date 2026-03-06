---
title: Publikuok Python paketą į PyPI su uv
description: Kaip sukurti, sukompiliuoti ir publikuoti Python paketą į PyPI naudojant uv
date: 2026-02-16T04:00:00Z
author: Ąžuolas Krušna
tags: ["Programų inžinerija", "Python", "uv", "PyPI"]
ShowCodeCopyButtons: true
draft: true
---

Labas! Šiandien pasidalinsiu kaip nuo nulio sukurti Python paketą ir jį publikuoti į PyPI naudojant [uv](https://docs.astral.sh/uv/) --- itin greitą Python paketų ir projektų valdymo įrankį, parašytą Rust kalba.

Po šio straipsnio turėsi savo Python paketą, kurį bet kas galės įsidiegti su `pip install tavo-paketas` arba `uv add tavo-paketas`.

### Kas yra uv?

[uv](https://github.com/astral-sh/uv) --- tai vienas įrankis, pakeičiantis `pip`, `pip-tools`, `pipx`, `poetry`, `pyenv`, `twine`, `virtualenv` ir dar daugiau. Jį sukūrė [Astral](https://astral.sh) komanda, žinoma dėl [Ruff](https://github.com/astral-sh/ruff) linterio. uv yra 10--100 kartų greitesnis nei `pip` ir palaiko visą projekto gyvavimo ciklą --- nuo inicijavimo iki publikavimo.

### Kas yra PyPI?

[PyPI](https://pypi.org/) (angl. *Python Package Index*) --- tai oficiali Python paketų saugykla. Kai rašai `pip install requests` arba `uv add requests` --- paketas parsisiunčiamas būtent iš PyPI.

### 1. Įsidiegiame uv

Jeigu dar neturi uv, įsidiek jį terminale:

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Patikrink, kad veikia:

```bash
uv --version
```

### 2. Sukuriame naują biblioteką

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

Atkreipk dėmesį --- naudojama `src` struktūra (angl. *src layout*). Tai reiškia, kad tavo paketo kodas gyvena `src/` direktorijoje. Tokia struktūra užtikrina, kad paketas bus testuojamas iš instaliuotos versijos, o ne tiesiog iš projekto šaknies.

### 3. Peržvelgiame pyproject.toml

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

Čia matome dvi svarbias dalis:

- **`[project]`** --- paketo metaduomenys: pavadinimas, versija, aprašymas, priklausomybės.
- **`[build-system]`** --- kompiliavimo sistema. uv naudoja savo `uv_build` sistemą pagal nutylėjimą, tačiau galima naudoti ir `hatchling`, `flit-core`, `setuptools` ar kitas.

Papildyk metaduomenis pagal savo poreikius --- aprašymą, autorių, licenciją:

```toml
[project]
name = "mano-paketas"
version = "0.1.0"
description = "Mano nuostabus Python paketas"
readme = "README.md"
requires-python = ">=3.11"
license = "MIT"
authors = [
    { name = "Tavo Vardas", email = "tavo@email.com" },
]
dependencies = []
```

### 4. Rašome kodą

Atidaryk `src/mano_paketas/__init__.py` ir parašyk savo paketo logiką. Pavyzdžiui:

```python
def sveiki(vardas: str = "Pasauli") -> str:
    return f"Labas, {vardas}!"
```

Galime iškart patikrinti, kad veikia:

```bash
uv run python -c "from mano_paketas import sveiki; print(sveiki('PyPI'))"
```

Turėtum pamatyti:

```
Labas, PyPI!
```

### 5. Pridedame priklausomybes (jei reikia)

Jeigu tavo paketas naudoja kitas bibliotekas, pridėk jas su `uv add`:

```bash
uv add requests
```

Tai automatiškai atnaujins `pyproject.toml` bei `uv.lock` failą.

### 6. Kompiliuojame paketą

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

- `.tar.gz` --- tai šaltinio (angl. *source*) distribucija su visu kodu.
- `.whl` --- tai sukompiliuota dvejetainė (angl. *wheel*) distribucija, kuri įsidiegia greičiau.

Publikuojant rekomenduojama naudoti `--no-sources` vėliavėlę, kad paketas teisingai kompiliuotųsi be uv-specifinių šaltinių nustatymų:

```bash
uv build --no-sources
```

### 7. Susikuriame PyPI paskyrą ir API raktą

Prieš publikuojant reikia:

1. Susikurti paskyrą [pypi.org](https://pypi.org/account/register/).
2. Susigeneruoti API raktą (angl. *API token*): eik į [Account Settings → API tokens](https://pypi.org/manage/account/#api-tokens) ir sukurk naują raktą.

Svarbiausia --- **PyPI nepalaiko slaptažodžio autentifikacijos**, todėl būtinas API raktas.

Rekomenduoju pirmiausia išbandyti su [TestPyPI](https://test.pypi.org/) --- tai PyPI kopija testavimui. Susikurk paskyrą ir API raktą ten atskirai.

### 8. Publikuojame į TestPyPI (rekomenduojama pirma)

Išbandyti galima su TestPyPI --- tai saugi aplinka, kur galima bandyti publikavimą be pasekmių.

Pirma, aprašyk TestPyPI indeksą savo `pyproject.toml`:

```toml
[[tool.uv.index]]
name = "testpypi"
url = "https://test.pypi.org/simple/"
publish-url = "https://test.pypi.org/legacy/"
explicit = true
```

Tuomet publikuok:

```bash
uv publish --index testpypi --token pypi-tavo-testpypi-api-raktas
```

Patikrink, ar paketas pasiekiamas:

```bash
uv run --with mano-paketas --index-url https://test.pypi.org/simple/ --no-project -- python -c "from mano_paketas import sveiki; print(sveiki())"
```

### 9. Publikuojame į PyPI

Kai viskas veikia su TestPyPI, publikuojame į tikrąjį PyPI:

```bash
uv publish --token pypi-tavo-pypi-api-raktas
```

Voilà! Tavo paketas dabar pasiekiamas visiems!

Patikrink, ar paketas teisingai įsidiegia:

```bash
uv run --with mano-paketas --no-project -- python -c "from mano_paketas import sveiki; print(sveiki())"
```

`--no-project` vėliavėlė užtikrina, kad paketas bus instaliuotas iš PyPI, o ne iš lokalios projekto direktorijos.

### 10. Atnaujiname versiją

Kai norėsi publikuoti naują versiją, uv turi patogią komandą:

```bash
# Nustatyti konkrečią versiją
uv version 1.0.0

# Pakelti minor versiją (0.1.0 → 0.2.0)
uv version --bump minor

# Pakelti patch versiją (0.1.0 → 0.1.1)
uv version --bump patch
```

Po versijos pakeitimo --- vėl `uv build` ir `uv publish`.

### Apibendrinimas

Štai ir viskas! Apžvelgėme visą kelią nuo tuščios direktorijos iki Python paketo PyPI saugykloje:

1. Įsidiegėme uv
2. Inicijavome biblioteką su `uv init --lib`
3. Sukonfigūravome `pyproject.toml`
4. Parašėme kodą
5. Sukompiliavome su `uv build`
6. Išbandėme su TestPyPI
7. Publikavome su `uv publish`

Visam tam nereikėjo nei `setuptools`, nei `twine`, nei `setup.py` --- viskas per vieną įrankį.

---

### Šaltiniai

- uv dokumentacija: Building and publishing a package https://docs.astral.sh/uv/guides/package/
- uv dokumentacija: Working on projects https://docs.astral.sh/uv/guides/projects/
- uv dokumentacija: Creating projects https://docs.astral.sh/uv/concepts/projects/init/
- PyPI: Python Package Index https://pypi.org/
- uv GitHub repozitorija https://github.com/astral-sh/uv

---

_"Tikras atradimo kelias --- tai ne naujų žemių ieškojimas, o matymas naujomis akimis."_ --- Marcel Proust
