+++
title = "Susikurk savo blogą su Hugo"
description = "Susikurk savo blogą su Hugo ir patalpink Netlify nemokamai"
date = 2021-01-20T03:00:00Z
author = "Ąžuolas Krušna"
tags = ["Programų inžinerija", "Susikurk savo blogą", "Hugo"]
+++

Čia dalinsiuosi procesu kaip susikurti blogą su Hugo. Perskaitę ir atlikę veiksmus turėsime blogą su internetiniu adresu nemokamai. Jeigu įdomu, skaityk toliau.

Dalinuosi ne tik instrukcijomis, bet ir savo pasakojimu, nuomone bei mintimis. Jeigu domina tik instrukcijos, daug trumpesnį anglišką variantą galite rasti čia [[1]](https://flaviocopes.com/start-blog-with-hugo/).

Instrukcijose yra du žingsniai, kurių neapžvelgiu, - vartotojų susikūrimas Netlify ir Github platformose, jeigu neturite. Tikiuosi, kad tai nesudarys kliūčių kuriantis savo blogą. Atlikti žingsnius taip pat pravers žinios apie versijavimo sistemas bei terminalą (komandinę eilutę). Minimalios - bloge šie žingsniai aprašyti, tačiau gali būti, kad per stambiai. Jei taip, tai parašykite - padėsiu ir parašysiu aiškiau.

Blogą norėjau turėti jau labai seniai. Kūriau jį tik pradėjęs programuoti, bet sutikau keletą kliūčių. Pasikūriau jį pagal vieną mokomąją medžiagą internete, taigi jis buvo labai paprastas ir man trūko žinių bei patirties pasidaryti jį mandresnį. Neturėjau žinių kur jį nemokamai patalpinti (host). Ir neparašiau nei vieno prasmingo posto, tik žaidžiau tikrai ne patį įdomiausią žaidimą - rašiau, tryniau, keičiau, rašiau, tryniau kodą, kuris aprašo tinklapio spalvas, elementų poziciją, dydį, išsidėstymą, interaktyvumą ir t.t. Patekau į tam tikrus programuotojo spąstus. Galbūt iš tikrųjų nenorėjau rašyti blogo, o tik pasikurti.

Nesu dabar tikras kaip tada galvojau, bet dabar žinau, kad turiu norą rašyti bei dalintis mintimis ir naudingomis žiniomis. Ir visgi man yra labai svarbu nepatekti į programuotojo spąstus kaip pirmu atveju - neužsižaisti iki begalybės poliruojant šį blogą. Jūs dabar jį matot - yra keletas adresų į Twitter, į el. paštą ir įvairiausių nereikalingų menkniekų, išdėstymas ir spalvų paletė nėra mano mėgstamiausia, bet žinau - bloge svarbiausia yra tekstas, nes jei jis neįdomus, tai visas blogas neįdomus ir visa kita niekam nerūpės. Ir žinot ką? Matau, kad pavyko. Iš tiesų buvo stiprus noras pasimodifikuoti tam tikrus dalykus pagal save, bet pagalvojau, kad nekeisiu specialiai, kartu tai padarysim - parodysiu ir jums.

Čia aš nieko naujo neišrandu. Apie Hugo išgirdau gerus atsiliepimus iš Chris Ferdinandi [[2]](https://gomakethings.com/static-websites/) ir neseniai pasitvirtinau iš Flavio Copes [[1]](https://flaviocopes.com/start-blog-with-hugo/). Tačiau pasidalinu visu šiuo procesu lietuviškai ir perkošiu per savo mintis.

Kodėl tai darau? Manau, kad Lietuvoje blogų kūrimas su Hugo yra dar labai nepopuliarus, ir manau, kad šios žinios yra naudingos. O juk galėjai tiesiog pasidalinti anglišku variantu. Kodėl lietuviškai? Manau, kad šia tema yra per mažai turinio lietuviškai, kad lietuviškas turinys gali pasiekti platesnę auditoriją ir būti dar labiau naudingas Lietuvos žmonėms. Kartu tai mane skatina pažiūrėti į visas internetines technologijas iš kitos pusės - surasti tinkamus lietuviškus vertimus internetiniams terminams tikrai reikia gerai palaužyti kaklą, o ne visuomet pavyksta (galbūt ir ne visuomet reikia), taigi rasite angliškų žargonų. Tikiuosi, kad jie nemaišys, o kaip tik padės suprasti esmę.

Žiniomis dalinuosi įkvėptas "Lėtas pelnas" blogo https://letaspelnas.lt bei vėliau atrastų kitų lietuviškų bei angliškų investavimo blogų, kurie man suteikė labai daug naudingų žinių ir idėjų (https://balticmustache.lt, https://www.honestfire.lt, [https://www.mrmoneymustache.com](https://www.mrmoneymustache.com)).

### Kodėl Hugo?

Iš kart pasakau, kad nesu dorai išbandęs kitų platformų, bet kaip programuotojas nenorėjau jokių platformų, kuriose galima susidėlioti blogą be kodo, ir nenorėjau turėti nieko bendro su duomenų baze.

Platformos, kuriose galima susidėlioti blogą be kodo nėra blogai ir galbūt ne programuotojui tai yra daugeliu atvejų daug geresnis kelias. Šis atvejis neprogramuojančiam žmogui gali būti tik sudėtingas ir visai be reikalo. Tačiau manau, kad pamankštinti smegenis ne savoj sferoj yra labai sveika, todėl jei pritari - *let's go for this ride*. Galiu pasakyti, kad Hugo yra parašytas ant visai naujos ir perspektyvios programavimo kalbos Go, kurios aš pats nemoku, todėl jeigu reikės kažką ten keisti - pasimokysiu ir aš. O dėl kodo dar norėjau pasakyti, kad būtinai noriu turėti visas laisves su kodu daryti ką tik užsimanęs ir nebūti priklausomas nuo platformos suteikiamų galimybių.

Nenorėjau naudoti pačios populiariausios tinklapių kūrimo sistemos - Wordpress. Iškart pasakau, kad tikrai neturiu nieko asmeniško prieš šią sistemą, jos vartotojus bei atstovus. Priešingai - labai smagu, kad yra tokia sistema, kuria pastatyta beveik pusė (39.5 %) visų internetinių puslapių [[3]](https://kinsta.com/blog/wordpress-statistics/). Internetas yra globali biblioteka, todėl Wordpress pagalba turime daug didesnę biblioteką, o kokia technologija pastatyta biblioteka (medinė ar betono blokas, iškilęs sovietmečiu) yra visiškai nesvarbu lyginant su jos turiniu. Tai labiau iš praktinės pusės - nenoriu sukti galvos kur patalpinti papildomai duomenų bazę ir dar galimai už ją mokėti. Iš tiesų manau, kad duomenų bazė gali papildomai pasitarnauti kuriant “gudresnį” blogą, nes su ja patogu transformuoti bei agreguoti duomenis, bet kartu atsiranda dar vienas lygis sistemų bendravime, o šitai noriu išvengti, jeigu galima. Duomenų bazė šiam blogui man nėra reikalinga - bent jau tai nematau dabar ar tolimoj ateityje, todėl per daug ir nesuku galvos.

Platformų, veikiančių panašiai kaip Hugo, yra ir daugiau [[4]](https://jamstack.org/generators/), tačiau jų čia nenagrinėsiu.

### Hugo privalumai

Remiantis šaltiniu [[1]](https://flaviocopes.com/start-blog-with-hugo/), Hugo yra nuobodus. O tai yra labai gerai, nes susikoncentruosim į turinį, o ne pikselių stumdymą - nepakliūsim į programuotojų spąstus. Jis nenaudoja jokios _fancy_ technologijos, kurių šiais laikais apstu. Tačiau kartu Hugo yra ir labai lankstus - galima susikurti įvairius paprastam blogui neįprastus dalykus. Galiausiai, Hugo yra greitas. Jo branduolys yra parašytas su Go, kuri yra žinoma kaip labai greita kalba.

Hugo viskas užsirašo *Markdown* [[5]](https://www.markdownguide.org/basic-syntax/). Man šis formatas labai patinka, nes yra labai paprastas ir funkcionalus, bet esu tikras, kad ne visiems jis gali patikti.

Rašyti blogą su Hugo reikia naudoti kodo versijavimą. Mes šiam reikalui naudosime Github. Vėliau yra naudojama tam tikra technologija, kuri sukelia šį kodą į serverį, jame sugeneruoja puslapius iš to kodo ir suteikia prie jų prieigą (per internetinį adresą).

### Puslapio talpinimas

Kadangi Hugo blogas yra visiškai statinis, mes galime pasinaudoti platformomis, kurios suteikia nemokamą tokių puslapių talpinimą: Github, Netlify, Vercel ir t.t. 

Vienintelis mokamas dalykas yra domenas, tačiau neužimtas domenas kainuoja apie 10 eurų per metus, tai manau, kad tikrai nebrangiai. Jeigu jau turite domeną, galite panaudoti jį. Tačiau jį nėra būtina turėti, paminėtos platformos suteiks jums domeną, tačiau veikiausiai ne tokį, kokio norite. Maniškė gavo toki adresą https://practical-goldwasser-df318e.netlify.app/

Kaip supratote, talpinsime Netlify platformoje.

### Įsirašom Hugo

Atsidarom terminalą (komandinę eilutę):

#### MacOS

*Terminal* programą ir tuomet rašom

```zsh
brew install hugo
```

Neturim brew? Įsirašom su šia komanda terminale

```zsh
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

#### Windows

*Command Prompt* programą ir rašom

```zsh
choco install hugo -confirm
```

Neturim choco? Įsirašom iš čia https://chocolatey.org/

#### Linux

Kaip įsirašyti galima rasti čia https://gohugo.io/getting-started/installing/

### Pasileidžiam Hugo

Kai tik įsirašom Hugo, paslapius galime sugeneruoti šia komanda terminale

```zsh
hugo new site myblog
```

Ši komanda sugeneruos naują `myblog` direktoriją ten, kur ją paleisit. Rekomenduoju jai susikurti dedikuotą direktoriją (pavyzdžiui `www`), tačiau tai nėra būtina. Kad žinotumėte kur esate, galite pasitikrinti su `pwd` komanda. Vaikščioti per direktorijas galime su komandomis: `cd ..` - atgal, `cd direktorijos_pavadinimas` - į direktorijos vidų. Aš savo blogo direktoriją įsidedu darbalaukio `Code` direktorijoje - `/Users/Azis/Dekstop/Code`.

### Renkamės blogo temą

Prieš pradedant rašyti mums dar reikia pasirinkti Hugo temą. Yra daugybė pasirinkimų https://themes.gohugo.io. Aš pagal Flavio Copes pavyzdį pasirinkau [https://themes.gohugo.io/ghostwriter/]( https://themes.gohugo.io/ghostwriter/ ). Vėliau mes ją padailinsime. Galite rinktis bet kurią, tačiau tolimesni blogo įrašai bus susiję su šia tema, todėl rekomenduoju rinktis ją.

Tuomet einam į https://github.com/jbub/ghostwriter/archive/master.zip ir parsisiunčiam dabartinę versiją. Išpakuojame ją į mūsų blogo direktoriją `themes/ghostwriter`.

![Tema Hugo direktorijoje](../theme.png)

Tada keliaujam į `exampleSite` direktoriją. Ten dar atsidarom `content` subdirektoriją. Matom `page`, `post` ir `project` direktorijas.

![Pavyzdiniai įrašai](../example_posts.png)

Nusikopijuojam `page` ir `post` į blogo `content` direktoriją.

![Įrašai](../posts.png)

### Konfigūracija

Pavyzdiniai įrašai taip pat turi pavyzdinį konfigūracijos `config.yaml` failą `themes/ghostwriter/exampleSite/config.toml`. Šis failas suteikia Hugo papildomos informacijos. Visgi nerekomenduoju jo kopijuoti, nes ten yra visko per daug. Vietoj to, galime panaudoti šitokią konfigūraciją:

```toml
baseurl = "/"
title = "My blog"
theme = "ghostwriter"

[Params]
    mainSections = ["post"]
    intro = true
    headline = "My headline"
    description = "My description"
    github = "https://github.com/XXX"
    twitter = "https://twitter.com/XXX"
    email = "XXX@example.com"
    opengraph = true
    shareTwitter = true
    dateFormat = "Mon, Jan 2, 2006"

[Permalinks]
    post = "/:filename/"
```

Vėliau galime ją pasikeisti kaip tik norėsime.

Tuomet leidžiame iš terminalo (būtinai iš blogo direktorijos)

```
hugo serve
```

![Blogo paleidimas](../serve.png)

Atsidarom naršyklėjė [http://localhost:1313](http://localhost:1313) ir turėtume matyti mūsų puslapį!

![Pradinis puslapis](../initial.png)

Čia matome mūsų pradinį puslapį. Įrašų sąrašas yra paimtas iš `content/post` direktorijos.

![Pradiniai įrašai](../initial_posts.png)

Atsidarome bet kurį iš jų ir keičiame. Išsaugojus svetainė automatiškai atnaujins turinį. Pavyzdžiui, susikuriame naują failą `post.md` su štai tokiu turiniu:

```md
+++
title = "Pirmas įrašas"
description = "Pirmas įrišimas"
date = 2021-01-27T10:13:50Z
author = "Adomas ir Ieva"
+++

# Labas

Labas, pasauli!
```

Išsaugom ir turi naują įrašą.

![Labas](../labas.png)

Nesudėtinga, ar ne?

### Talpinimas Netlify

Blogui talpinti pirmiausia susikuriu repozitoriją (kodo saugyklą) Github https://github.com/. Jeigu neturime, reikia susikurti vartotoją, parsisiųsti programą.

Tuomet iš _File_ meniu pasirenku "New Repository":

![Repozitorija](../repository.png)

Tokį patį vaizdą galime gauti tiesiog nutempdami `myblog` direktoriją į Github aplikaciją. Suteikiu savo repozitorijai `myblog` pavadinimą (Name) ir nurodau iki jos atitinkamą kelią (Local Path). Spaudžiu "Create Repository". Tai automatiškai padaro patį pirmąjį commit. 

![Pirmasis commit](../initial_commit.png)

Tada spaudžiu "Publish repository" mygtuką - nusiunčiu ją į Github. Repozitoriją galime pasirinkti privačią arba viešą. Aš pasirenku viešą, tačiau jūs galite rinktis ir privačią, žinoma.

![Github publish](../github_publish.png)

Kai tik turime repo (repozitoriją) Github,

![Github](../github.png)

galime keliauti į Netlify https://www.netlify.com/. Tam reikės susikurti Netlify vartotoją. Netlify spaudžiu “New site from Git” mygtuką. Pasirenku Github.

![Naujas puslapis](../new_site.png)

Suteikiu Netlify priėjimą prie mano blogo repo ir ją pasirenku.

![Netlify repo](../netlify_repo.png)

Netlify šią repo automatiškai identifikavo kaip Hugo repo, taigi automatiškai parinko jai sugeneravimo komandą (`hugo`).

![Netlify build](../netlify_build.png)

Spaudžiame "Deploy Site" ir Netlify pradeda talpinimo procesą. Po šio veiksmo turime veikiantį blogą online!

### Jei nepavyko

Nieko! Man irgi nepavyko iš pirmo karto.

![Deploy fail](../deploy_fail.png)

Sėkmingai patalpinti Netlify pavyko tik iš trečio karto. Antrą kartą bandžiau talpinti tikėdamasis, kad susitvarkys. _Nope_. Tuomet ėjau skaityti logus, kad suprasčiau kas negerai. 

![Deploy fail logai](../deploy_fail_logs.png)

Pasirodo, Netlify naudoja netinkamą Hugo versiją mano kodui. Pažiūrėjau kokią versiją turiu lokaliai su komanda `hugo version`.

![Hugo lokali versija](../hugo_version_local.png)

 
Nurodau Netlify, kad ją naudotų. Spaudžiu "Site settings" -> "Build & Deploy" -> "Environment" -> "Environment variables". Tuomet ten įvedu `HUGO_VERSION` kintamąjį ir suteikiu jam reikšmę `0.79.0`.

![Hugo versija](../hugo_version.png)

Sugrįžtu prie talpinimų, paleidžiu dar kartą ir _voilà_ - pavyko!

Jei ištinka kiti trukdžiai - einam skaitom logus, googlinam ir išsprendžiam.

### Pavyko

Turėtumėte iškart gauti internetinį adresą. Aš gavau https://practical-goldwasser-df318e.netlify.app/

### Kaip atsiranda bloge nauji įrašai?

Kai tik pakeičiam vieną iš įrašų ar pridedam naują Github, Netlify automatiškai atnaujina blogą pergeneruodamas ir pertalpindamas jį. Keisti kodą galime lokaliai ir tada nusiųsti į Github arba galima tai daryti tiesiai Github - taip galima rašyti postus ne tik per kompiuterį, bet ir telefoną ar planšetę. Atkreipkite dėmesį į mano ekraną - internetinio adreso kelią - tai turėtų pagelbėti nueiti iki norimo failo ir jį pakeisti.

![Github redagavimas](../github_edit.png)

Pabaigoj norėčiau pasidžiaugti, kad Github bei Netlify suteikia galimybę nemokamai naudotis jų serveriais, nes be jų čia nieko nebūtų.

---

[Čia](https://www.aziogas.lt/uzsidek-domena-savo-blogui-ant-netlify/) patikrink kaip užsidėti domeną savo Hugo blogui!

_May the force be with you_
