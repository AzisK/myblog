+++
title = "Nuspalvink savo terminalą trispalve"
description = "Nuspalvink savo terminalą trispalve artėjant Vasario 16"
date = 2021-02-12T02:13:50Z
author = "Ąžuolas Krušna"
+++

# Nuspalvink savo terminalą trispalve

Programuotojau, kviečiu Tave artėjant Vasario 16 - Lietuvos nepriklausomybės paskelbimo proga - nusispalvinti terminalą trispalve. Gražios spalvos, gražus terminalas - maloniau dirbti.

Gidas kol kas pritaikytas tik MacOS vartotojams.

Atlikę veiksmus galėsim grožėtis tokiu terminalu

![Lietuva zsh](../lithuania-zsh.png)

### Naudoju bash shell

Tuomet lendu į `~/.bash_profile` ir ten pasirašau eilutę:
```
export PS1="\e[0;33m\u\e[m\e[0;32m@\e[m\e[0;31m\W\e[m[\t]\$ "
```

Atsidarau terminalą per naują ir matau trispalvę

![Lietuva bash](../lithuania-bash.png)

Šį terminalą naudoju darbo kompiuteryje jau kelerius metus - daug smagiau dirbti negu su įprastu.

### Naudoju zsh 

1. Tuomet, jeigu neturiu, įsirašau "Oh My Zsh" https://ohmyz.sh/
2. Einu į `~/.oh-my-zsh/themes` ir ten susikuriu temos failą (pasikūriau `lithuania.zsh-theme`)
3. Į jo turinį įrašau visa šitai
```zsh
right_triangle() {
   echo $'\u276f'
}

prompt_indicator() {
   echo $'%B\u276f%b'
}

arrow_start() {
   echo "%{$FG[$ARROW_FG]%}%{$BG[$ARROW_BG]%}%B"
}

arrow_end() {
   echo "%b%{$reset_color%}%{$FG[$NEXT_ARROW_FG]%}%{$BG[$NEXT_ARROW_BG]%}$(right_triangle)%{$reset_color%}"
}

ok_username() {
   ARROW_FG="016"
   ARROW_BG="226"
   NEXT_ARROW_BG=""
   NEXT_ARROW_FG="226"
   echo "$(arrow_start) %n $(arrow_end)"
}

err() {
  echo "%{$FG[160]✘%}"
}

err_username() {
   ARROW_FG="016"
   ARROW_BG="226"
   NEXT_ARROW_BG=""
   NEXT_ARROW_FG="226"
   echo "$(err) $(arrow_start) %n $(arrow_end)"
}

# return err_username if there are errors, ok_username otherwise
username() {
   echo "%(?.$(ok_username).$(err_username))"
}

directory() {
   ARROW_FG="016"
   ARROW_BG="034"
   NEXT_ARROW_BG=""
   NEXT_ARROW_FG="034"
   echo "$(arrow_start) %2~ $(arrow_end)"
}

current_time() {
   echo "%{$fg[white]%}%*%{$reset_color%}"
}

git_prompt() {
   ARROW_FG="016"
   ARROW_BG="196"
   NEXT_ARROW_BG=""
   NEXT_ARROW_FG="196"
   echo "$(arrow_start) $(git_prompt_info) $(arrow_end)"
}

# set the git_prompt_info text
ZSH_THEME_GIT_PROMPT_PREFIX=""
ZSH_THEME_GIT_PROMPT_SUFFIX=""
ZSH_THEME_GIT_PROMPT_DIRTY="*"
ZSH_THEME_GIT_PROMPT_CLEAN=""

# set the git_prompt_status text
ZSH_THEME_GIT_PROMPT_ADDED="%{$fg[cyan]%} ✈%{$reset_color%}"
ZSH_THEME_GIT_PROMPT_MODIFIED="%{$fg[yellow]%} ✭%{$reset_color%}"
ZSH_THEME_GIT_PROMPT_DELETED="%{$fg[red]%} ✗%{$reset_color%}"
ZSH_THEME_GIT_PROMPT_RENAMED="%{$fg[blue]%} ➦%{$reset_color%}"
ZSH_THEME_GIT_PROMPT_UNMERGED="%{$fg[magenta]%} ✂%{$reset_color%}"
ZSH_THEME_GIT_PROMPT_UNTRACKED="%{$fg[white]%} ✱%{$reset_color%}"

PROMPT='$(username)$(directory)$(git_prompt) '
RPROMPT='$(git_prompt_status) $(current_time)'
```
4. Lendu į failą `~/.zshrc` ir nustatau `ZSH_THEME=lithuania`
5. Atsidarau terminalą iš naujo
6. Grožiuosi

![Lietuva zsh](../lithuania-zsh.png)

### Jei nepavyko

Gali būti, kad mūsų zsh nesupras tam tikrų simbolių, pavyzdžiui, rodyklyčių. Tuomet reikėtų nueiti į failą `~/.zshrc` ir ten atkomentuoti šią eilutę (82)

```zsh
# You may need to manually set your language environment
export LANG=en_US.UTF-8
```

***

Terminalą spalvinau pagal šį gidą https://blog.carbonfive.com/writing-zsh-themes-a-quickref/

Gražios ir laisvos šventės!

_May the force be with you_
