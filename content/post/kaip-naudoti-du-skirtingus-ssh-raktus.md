---
title: Kaip naudoti du skirtingus SSH raktus
description: Kaip naudoti du skirtingus SSH raktus dirbant su asmenine ir darbo repozitorijomis
date: 2026-01-26T04:00:00Z
author: Ąžuolas Krušna
tags: ["Programų inžinerija", "Git", "GitHub", "SSH"]
ShowCodeCopyButtons: true
---

Tai galime pasiekti apsirašant kokius raktus kada naudoti `~/.ssh/config` faile.

Pirmiausia, mums reikia įsitikinti, kad turime raktus savo kompiuteryje bei kodo versijavimo (toliau kaip pavyzdį naudosime tik Github, tačiau šis gidas tinka ir kitom git kodo versijavimo platformoms) svetainėje.

### Jeigu neturime raktų, galime juos susikurti

```zsh
# Asmeninis raktas
ssh-keygen -t ed25519 -C "your@personal.email" -f ~/.ssh/id_ed25519_personal

# Darbo raktas
ssh-keygen -t ed25519 -C "your@work.email" -f ~/.ssh/id_ed25519_work
```

Tai mums sukurs 4 failus `~/.ssh` direktorijoje:
- `id_ed25519_personal` (privatus raktas)
- `id_ed25519_personal.pub` (viešas raktas)
- `id_ed25519_work` (privatus raktas)
- `id_ed25519_work.pub` (viešas raktas)

### Pridedame raktus į SSH agentą

Šis žingsnis nėra būtinas, jeigu kaip viršuje nenaudojamas _passphrase_.

```zsh 
ssh-add ~/.ssh/id_ed25519_personal
ssh-add ~/.ssh/id_ed25519_work
```

### Įsitikiname, kad turime raktus savo paskyroje

Prisijungę į Github svetainės paskyrą, įsitikiname, kad ten turime šiuos raktus. Tiek asmeninėj, tiek darbo. Jeigu neturime, prisidedame.

Taigi, nusikopijuojame asmeninį viešą raktą.

```zsh
# Nukopijuojame 
pbcopy < ~/.ssh/id_ed25519_personal.pub
```

Tuomet einame į GitHub ir spaudžiame ant savo vartotojo ikonėlės (Open user navigation menu) → Settings → SSH and GPG keys → New SSH key ir įklijuojame.

![Github add new ssh key](../github-add-new-ssh-key.png)

Tą patį atliekame su darbo raktu (`id_ed25519_work.pub`) darbo GitHub paskyroje.

### Susikonfigūruojame ~/.ssh/config failą

Jeigu neturime, susikuriame `~/.ssh/config` failą (`touch ~/.ssh/config`). Jeigu turime, atnaujiname taip:

```zsh
# Darbo paskyra (numatytoji)
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
    IdentitiesOnly yes

# Asmeninė paskyra
Host personal.github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal
    IdentitiesOnly yes
```

### Kaip tai veikia?

SSH konfigūracija leidžia sukurti _Host_ _alias_. Kai rašome `personal.github.com`, SSH žino, kad iš tikrųjų jungsis prie `github.com` (HostName), bet naudos asmeninį raktą (`IdentityFile`).

- **Host** --- tai _alias_, kurį naudosime git komandose
- **IdentitiesOnly yes** --- nurodo naudoti tik nurodytą raktą, ne visus iš SSH agento

### Kaip klonuoti repozitorijas?

Dabar klonuojant reikia naudoti atitinkamą Host alias. 

Jeigu iš Github, tai klonuojame adresą, kurį randame paspaudę Code → SSH

![Github clone ssh](../github-clone-ssh.png)

#### Darbo repozitorijai

Numatytąjai darbo repozitorijai naudojame jį ir nieko nekeičiame (naudojame įprastą `github.com`).

```zsh
git clone git@github.com:work-user/work-project.git
```

#### Asmeninei repozitorijai

O klonuojant asmeninę repozitoriją, mums reikia pakeisti _host_ į `personal.github.com` _alias_.

```zsh
git clone git@personal.github.com:personal-user/personal-project.git
```

### Kaip pakeisti jau esančios repozitorijos alias?

Jeigu jau turime nusiklonavę repozitoriją ir norime teisingai sukonfigūruoti SSH raktus, tada mums reikia pakeisti nuotolinį repozitorijos adresą.

Patikriname esamą nuotolinį adresą (URL).
```zsh
git remote -v
```

Pakeičiame _alias_ į asmeninį.
```zsh
git remote set-url origin git@personal.github.com:personal-user/personal-project.git
```

### Ar tikrai veikia?

Ar SSH raktų konfigūracija tikrai veikia, galime patikrinti taip:

```zsh
# Tikrina asmeninę paskyrą
ssh -T git@personal.github.com
```

```zsh
# Tikrina darbo paskyrą
ssh -T git@github.com
```

Jeigu matome kažką panašaus į šią žinutę, nadinasi veikia!

Hi AzisK! You've successfully authenticated, but GitHub does not provide shell access.

---

Voilà! Dabar mums nebereikės sukti galvos kur koks SSH raktas bus naudojamas, nes tai automatizavome!

Tačiau nepamirštame, kad SSH raktai tik autentifikuoja mus prie GitHub, bet _commit_ autorystė nustatoma pagal git vartotojo vardą ir el. paštą. Kad _commit'ai_ rodytų teisingą autorių, turime sukonfigūruoti ir šiuos nustatymus.

Kaip tai padaryti esu aprašęs šiame straipsnyje straipsnyje [Kaip naudoti du skirtingus git vartotojus](/kaip-naudoti-du-skirtingus-git-vartotojus/).

### Šaltiniai

- GitHub Docs: Generating a new SSH key https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent
