---
title: "Įsidėk kodo kopijavimo mygtuką savo bloge"
date: 2023-11-07T22:41:24+01:00
ShowCodeCopyButtons: true
draft: true
---

Labas! Tai dar vienas straipsnis iš [Susikurk savo blogą su Hugo](https://www.aziogas.lt/susikurk-savo-bloga-su-hugo/) serijos. Čia mes įsidėsime kopijavimo mygtuką kodo blokams mūsų straipsniuose. 

Kadangi rašau blogą su daug techninių detalių ir kodo pavyzdžių, tai neabejoju, jog tai bus pravartu ne tik man, bet ir skaitytojui.

Šią inžinerinę kelionę pradedu Google paieška, nes manau, kad kažkas tikrai tai jau yra daręs. Akurat. Pasirodo, mano blogo naudojama tema Papermod net jau turi savy šį funkcionalumą! Man tereikia perduoti blogo įrašui `ShowCodeCopyButtons` kintamąjį kaip teisingą būlį `true`.

```yaml
---
ShowCodeCopyButtons: true
---
```

Paskutiniam rašytam straipsniui visi parametrai atrodytų taip

```yaml
---
title: "Docker sąvokos lietuviškai"
description: "Docker sąvokų apsibrėžimas lietuviškai bei jų tarpusavio ryšys"
date: 2023-10-25T04:00:00Z
author: "Ąžuolas Krušna"
tags: ["Programų inžinerija", "Lietuvių kalba", "Konteineris", "Docker", "Software", "Lithuanian language", "Container"]
ShowCodeCopyButtons: true
---
```

Dabar pelyte nuriedėjus ant kodo bloko matysime "copy" mygtuką, kurį paspaudus bus nukopijuotas visas kodo blokas! 

![Docker](../code_copy_button.png)

Kitas būdas prisidėti šį kopijavimo mygtuką yra įsirašyti šį parametrą į blogo konfigūraciją `config.toml`. Pliusas, kad tai bus automatiškai sugeneruojama visiems blogo straipsniams! Kartą įrašėm ir galime užmiršti, ir manau, kad tai yra labai geras sprendimas blogui.

```toml
[Params]
    ShowCodeCopyButtons = true
```

Minimalus minusas yra tai, kad šį funkcionalumą turės net ir straipsniai be kodo blokų. Tai reiškia, kad tie puslapiai turės ir minimaliai ilgesnį HTML failą, ir daugiau Javascript kodo.

Kol kas renkuosi perduoti šį kintamąjį tik tiems straipsniams, kuriuose jis reikalingas. Visgi jeigu ateityje matysiu, kad tai prižiūrėti reikalauja per daug investicijų, tuomet įjungsiu šią funkciją visiems straipsniams.

Džiaugiuosi, kad ši Hugo tema turi kodo blokų kopijavimo funkciją ir ją galime tiesiog įjungti.

### Kas, jeigu jūsų blogas neturi šio funkcionalumo?

Tuomet reikės jį susikurti. Jis gali būti įgyvendintas įvairiais būdais. Čia pasižiūrėsime kaip tai yra padaryta Hugo PaperMod temoje.

Radame, kad temos failas `footer.html` (themes/PaperMod/layouts/partials/footer.html) turi šį Javascript logikos prijungimą prie savo HTML, jei atitinka tam tikrą "if" salygą. https://github.com/adityatelange/hugo-PaperMod/blob/72ab73ffe5ba2354f71e419321134fd5f9cfd098/layouts/partials/footer.html#L87C1-L135C11

```HTML
{{- if (and (eq .Kind "page") (ne .Layout "archives") (ne .Layout "search") (.Param "ShowCodeCopyButtons")) }}
<script>
    document.querySelectorAll('pre > code').forEach((codeblock) => {
        const container = codeblock.parentNode.parentNode;

        const copybutton = document.createElement('button');
        copybutton.classList.add('copy-code');
        copybutton.innerHTML = '{{- i18n "code_copy" | default "copy" }}';

        function copyingDone() {
            copybutton.innerHTML = '{{- i18n "code_copied" | default "copied!" }}';
            setTimeout(() => {
                copybutton.innerHTML = '{{- i18n "code_copy" | default "copy" }}';
            }, 2000);
        }

        copybutton.addEventListener('click', (cb) => {
            if ('clipboard' in navigator) {
                navigator.clipboard.writeText(codeblock.textContent);
                copyingDone();
                return;
            }

            const range = document.createRange();
            range.selectNodeContents(codeblock);
            const selection = window.getSelection();
            selection.removeAllRanges();
            selection.addRange(range);
            try {
                document.execCommand('copy');
                copyingDone();
            } catch (e) { };
            selection.removeRange(range);
        });

        if (container.classList.contains("highlight")) {
            container.appendChild(copybutton);
        } else if (container.parentNode.firstChild == container) {
            // td containing LineNos
        } else if (codeblock.parentNode.parentNode.parentNode.parentNode.parentNode.nodeName == "TABLE") {
            // table containing LineNos and code
            codeblock.parentNode.parentNode.parentNode.parentNode.parentNode.appendChild(copybutton);
        } else {
            // code blocks not having highlight as parent class
            codeblock.parentNode.appendChild(copybutton);
        }
    });
</script>
{{- end }}
```

Čia mes į pačią logiką nesigilinsime, tačiau užmetus trumpą žvilgsnį galime suprasti, kad kiekvienam kodo blokui pagal `pre > code` selektorių yra įdedamas kopijavimo mygtukas. Galime įsidėti tai į savo blogą ir galbūt nieko nekeitus tai veiks. Jeigu neveiks, tuomet tai bus pavyzdys ir įkvėpimas pakeitimams.

Kartu šiam funkcionalumui yra reikalingas yra CSS kodas. Jis yra aprašytas faile `main.css` (themes/PaperMod/assets/css/common/main.css) https://github.com/adityatelange/hugo-PaperMod/blob/72ab73ffe5ba2354f71e419321134fd5f9cfd098/assets/css/common/main.css#L52-L68.

```CSS
.copy-code {
    display: none;
    position: absolute;
    top: 4px;
    right: 4px;
    color: rgba(255, 255, 255, 0.8);
    background: rgba(78, 78, 78, 0.8);
    border-radius: var(--radius);
    padding: 0 5px;
    font-size: 14px;
    user-select: none;
}


div.highlight:hover .copy-code,
pre:hover .copy-code {
    display: block;
}
```

 ### Keičiam kopijavimo mygtuką

 _Bus pratęsta_

Šaltiniai: 

- PaperMod | Features | Code Copy Button  https://github.com/adityatelange/hugo-PaperMod/wiki/Features#code-copy-button
