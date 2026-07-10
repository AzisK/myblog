---
title: Automatiškai aktyvuok Python venv terminale
description: Kaip automatiškai aktyvuoti Python virtualią aplinką (venv) terminale įžengus į direktoriją
date: 2026-07-10T08:00:00Z
author: Ąžuolas Krušna
tags: ["Programų inžinerija", "Python", "venv", "Terminalas", "Ghostty", ]
ShowCodeCopyButtons: true
---

> #### pi; n
```zsh
# ~/.zshrc
autoactivate_venv() {
  if [[ -d ".venv" && -f ".venv/bin/activate" ]]; then
    source .venv/bin/activate
  elif [[ -d "venv" && -f "venv/bin/activate" ]]; then
    source venv/bin/activate
  elif [[ -n "$VIRTUAL_ENV" ]]; then
    parentdir="$(dirname "$VIRTUAL_ENV")"
    if [[ "$PWD" != "$parentdir"* ]]; then
      deactivate
    fi
  fi
}

autoload -U add-zsh-hook
add-zsh-hook chpwd autoactivate_venv
autoactivate_venv
```

Labas! Šiame gide išmoksime automatiškai aktyvuoti virtualią Python aplinką vos tik įžengus į direktoriją. Visa tai padaryti labai paprasta pasinaudojus kabliais (_angl_. hooks), kurie aptinka direktorijos pasikeitimo įvykį, tačiau atrodys ir veiks kiek skirtingai priklausomai nuo to, kokią kriauklę (_angl_. shell) naudojame savo terminalui. Aš naudoju zsh, todėl pirmiausia apžvelgsiu tai.

Šiuo metu naudoju Ghostty terminalą ir man jis patinka ir iš pradžių maniau, kad reikės aiškintis kaip tai padaryti būtent Ghostty terminale, tačiau galiausiai radau, kad nepriklausomai nuo terminalo, mes tai galime atlikti kriauklės lygmeny.

### Kaip tai veikia?

Pirmiausia, tai zsh turi `chpwd` įvykį, kuris iššaunamas kaskart pakeitus direktoriją su `cd` komanda. Tuomet mes užregistruosime funkciją, kuri kaskart patikrins, ar dabartinėje direktorijoje yra virtuali aplinka (venv), o jeigu yra, tai ją aktyvuos. Kartu įdėsime į funkciją logiką, kuri neaptikus direktorijoje virtualios aplinkos ją deaktyvuos.

Ghostty šiuo atveju nieko ypatingo nedaro, jis tik paleidžia kriauklę (zsh), kuris nuskaito `~/.zshrc`, todėl tai ir veikia.

### Konfigūracija

Tai veikia ne tik su zsh. Mes galime tai pasidaryti bet kurioje kitoje kriauklėje: zsh, bash ar fish. Galiausiai skirsis tik kablių mechanizmas ir sintaksė.

#### Zsh

Į savo `~/.zshrc` failą pridedame šį kodą:

```zsh
autoactivate_venv() {
  if [[ -d ".venv" && -f ".venv/bin/activate" ]]; then
    source .venv/bin/activate
  elif [[ -d "venv" && -f "venv/bin/activate" ]]; then
    source venv/bin/activate
  elif [[ -n "$VIRTUAL_ENV" ]]; then
    parentdir="$(dirname "$VIRTUAL_ENV")"
    if [[ "$PWD" != "$parentdir"* ]]; then
      deactivate
    fi
  fi
}

autoload -U add-zsh-hook
add-zsh-hook chpwd autoactivate_venv
autoactivate_venv
```

Kaip tai vyksta?

1. Zsh turi `chpwd` kablį, o jis iššaunamas kaskart pakeitus direktoriją
2. Funkcija tikrina, ar egzistuoja `.venv` arba `venv` direktorija ir joje failas `bin/activate`
3. Jei venv aktyvus, bet išėiname iš projekto direktorijos, tuomet iškviečiamas `deactivate`
4. Paskutinė eilutė `autoactivate_venv` paleidžia funkciją iš karto atsidarius kriauklę, jeigu atsidarome ją iš karto projekte

#### Bash

Į savo `~/.bashrc` failą pridedame šį kodą:

```bash
autoactivate_venv() {
  if [[ -d ".venv" && -f ".venv/bin/activate" ]]; then
    source .venv/bin/activate
  elif [[ -d "venv" && -f "venv/bin/activate" ]]; then
    source venv/bin/activate
  elif [[ -n "$VIRTUAL_ENV" ]]; then
    parentdir="$(dirname "$VIRTUAL_ENV")"
    if [[ "$PWD" != "$parentdir"* ]]; then
      deactivate
    fi
  fi
}

PROMPT_COMMAND="autoactivate_venv;$PROMPT_COMMAND"
```

Bash neturi `chpwd`, todėl naudojamas `PROMPT_COMMAND`, o jis vykdomas prieš kiekvieną užklausą. Veikimo mechanizmas panašus kaip ir zsh.

#### Fish

Į `~/.config/fish/conf.d/venv.fish` failą pridedame šį kodą:

```zsh
function autoactivate_venv --on-variable PWD
    if test -d .venv; and test -f .venv/bin/activate.fish
        source .venv/bin/activate.fish
    else if test -d venv; and test -f venv/bin/activate.fish
        source venv/bin/activate.fish
    else if set -q VIRTUAL_ENV
        set parentdir (dirname $VIRTUAL_ENV)
        if not string match -q "$parentdir*" $PWD
            deactivate
        end
    end
end

autoactivate_venv
```

Fish naudoja `--on-variable PWD` funkciją, o ji iškviečiama, kai pasikeičia darbo direktorija. Atkreipiame dėmesį, kad Fish naudoja `activate.fish` vietoj `activate`.

#### Palyginimas

| Kriauklė | Kablys | Konfigūracijos failas |
|-------|-----------------|----------------------|
| Zsh | `chpwd` hook | `~/.zshrc` |
| Bash | `PROMPT_COMMAND` | `~/.bashrc` |
| Fish | `--on-variable PWD` | `~/.config/fish/conf.d/venv.fish` |

Ghostty veikia su visais trim, nes jis tiesiog paleidžia tavo naudojamą kriauklę.

### Naudojimas

Nepamirštame perkrauti kriauklę!

```zsh
source ~/.zshrc
```

Kitas būdas perkrauti yra atsidaryti naują sesiją.

Taigi, susikuriame so projekte virtualią aplinką!

```zsh
cd ~/Work/my-project
python3 -m venv .venv
```

Dabar kaskart, kai įeisime į šią direktoriją, virtuali aplinka aktyvuosis automatiškai. Kai išeisime, ji deaktyvuosis.

```zsh
cd ~/Work/my-project
# (.venv) atsiranda prieš užklausą ✓

cd ~
# (.venv) dingsta ✓
```

### Šaltiniai

- Zsh Hook Functions https://zsh.sourceforge.io/Doc/Release/Functions.html#Hook-Functions
- Python venv documentation https://docs.python.org/3/library/venv.html
