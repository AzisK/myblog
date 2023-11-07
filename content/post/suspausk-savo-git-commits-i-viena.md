---
title: "Suspausk savo git commits į vieną"
description: "Suspausk savo git commits į vieną įvairiais būdais"
date: 2023-06-03T03:00:00Z
author: "Ąžuolas Krušna"
tags: ["Programų inžinerija", "Git"]
ShowCodeCopyButtons: true
---

Angliškai veiksmas, suspaudžiant kelis git commits į vieną, vadinamas "squash". Jis naudojamas palaikyti tvarką bei pokyčių aiškumą git istorijoje sutraukiant to paties koncepto commits į vieną. Tai kartu sumažina nereikšmingų commits žinučių kiekį, kuris maišytų perprasti: kodo raidą ir priimtų sprendimų priežastis.

Tai galima paprastai padaryti naudojantis įvairioms kodo versijavimo programomis: Github Desktop, Pycharm CE, IntelliJ CE (CE yra community edition trumpinys, t.y. sukurta bendruomenėms, nemokama versija) etc. 

Žemiau galime matyti kaip patogiai galima suspausti 2 git commits į vieną Github Desktop bei 3 IntelliJ programose. Jei mums tiek ir tereikia, tai galime paieškoti šios funkcijos mūsų programoje, ja pasinaudoti ir eiti pasidžiaugti likusia diena.

2 commits suspaudimas Github.
![Squash commits Github](../squash_commits_github.png)

3 commits suspaudimas IntelliJ. Šiuo atveju prieš tai reikės būtinai nuimti šakai apsaugą (protected branch).
![Squash commits IntelliJ](../squash_commits_intellij.png)

Jeigu tokios galimybės nėra ar esame labai smalsūs ir žingeidūs, tuomet toliau gilinamės kaip tai atlikti naudojantis terminalu.

### Kaip tai padaryti naudojantis terminalu?

Tai padaryti su git CLI mums reikės atsiminti kiek paskutinių commits mes norime suspausti į vieną. Išsiaiškinę commits kiekį rašome į terminalą:

```zsh
git rebase --soft HEAD~<KIEKIS>
```

Pavyzdžiui, 5 paskutiniai mano commits buvo bug taisymas. Šių žinučių aš nei skaityti, nei matyti nenoriu, tad noriu kodo pokyčius prijungti prie paskutinio 6-to commit. Tuomet rašau:

```zsh
git rebase --soft HEAD~6
```

O tuomet jau suformuluoju atitinkamą git commit žinutę.

```zsh
git commit 'Naujas straipsnis "Suspausk savo git commits į vieną"'
```

Viskas, mes šiuos commits sutraukėme į vieną savo kompiuteryje.

### Kas, jeigu šie 6 mano commits jau yra remote repozitorijoje?

Tuomet mums reikės panaudoti jėgą, kuri perrašys git istoriją, nes lokalūs pokyčiai skiriasi nuo istorijos remote repozitorijoje.

Iš istorijos galime pasimokyti, kad istorija yra perrašoma propagandos tikslais, ir kad daugeliu atveju tai yra labai blogai. Visgi git istorija yra visai kas kita, tai kodo istorija, ji perrašoma tvarkos ir aiškumo tikslais ir taip tik kondensuojama. Ji taip pat yra decentralizuota. Jei prie šio kodo dirba daugiau žmonių, tai jie turės paskutines, priešpaskutines kodo versijas pas save net jeigu ir remote repozitorija pakeitė savo istoriją.

Tuomet mums belieka paleisti šią komandą

```zsh
git push --force
```

Yay! Dabar niekas nebematys mano žioplų klaidų!

### Kaip suspausti ne iš eilės einančius commits?

Tai atlikti galime naudojantis 

```zsh
git rebase --interactive HEAD~<KIEKIS>
```

Pavyzdžiui, norime apjungti paskutinį 1-ą ir 4-ą commits. Tuomet rašome

```zsh
git rebase --interactive HEAD~<4>
```

ir gauname tokią žinutę.

```zsh
pick 11111 Commit1
pick 22222 Commit2
pick 33333 Commit3
pick 44444 Commit4

# Rebase 11111..44444 onto 12345 (4 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup [-C | -c] <commit> = like "squash" but keep only the previous
#                    commit's log message, unless -C is used, in which case
#                    keep only this commit's message; -c is same as -C but
#                    opens the editor
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
#         create a merge commit using the original merge commit's
#         message (or the oneline, if no original merge commit was
#         specified); use -c <commit> to reword the commit message
# u, update-ref <ref> = track a placeholder for the <ref> to be updated
#                       to this position in the new commits. The <ref> is
#                       updated at the end of the rebase
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
```

Dabar mes perkelsime 4-tą commit į 2 vietą ir pakeisime "pick" į "squash". Tai suspaus 1-ą ir 4-ą commits į vietą. Galutinė mūsų žinutė atrodo taip:

```zsh
pick 11111 Commit1
squash 44444 Commit4
pick 22222 Commit2
pick 33333 Commit3
```

Jeigu šie 4 commits jau buvo remote repozitorijoje, tai mums ir vėl reikės panaudoti jėgą (force).

```zsh
git push --force
```

Dabar galime eiti miegoti ramia galva.

***

IntelliJ programa taip pat turi interaktyvaus rebase funkciją.
![Interactive rebase IntelliJ](../interactive_rebase_intellij.png)

***

Ar nori paskaityti kaip atlikti "pull-request squash ir merge" terminale? Skaityki tai [čia](https://www.aziogas.lt/suspausk-git-commits-is-pull-request-i-viena/)!

***

Šiai medžiagai parašyti buvo pasinaudota [Stack Overflow](https://stackoverflow.com/a/3921724/7714279), kolegos patarimai ir paties atradimai.

_May the force be with you_
