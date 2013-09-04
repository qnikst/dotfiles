#Set LS_COLORS
if [[ -f ~/.dir_colors ]]; then
        eval `dircolors -b ~/.dir_colors`
else
        eval `dircolors -b /etc/DIR_COLORS`
fi

# History settings
HISTFILE=~/.histfile
HISTSIZE=3000
SAVEHIST=3000
setopt append_history inc_append_history
setopt no_hist_beep
setopt hist_ignore_dups hist_ignore_all_dups hist_expire_dups_first
setopt hist_save_no_dups hist_find_no_dups
# Use the same history file for all sessions
setopt share_history
# Let the user edit the command line after history expansion (e.g. !ls) instead of immediately running it
setopt hist_verify

setopt extended_glob
setopt noequals
setopt nobeep
setopt correct
setopt autocd
setopt nohup
setopt hash_cmds

# Don't fail on unsuccessful globbing
unsetopt nomatch

bindkey -v

# The following lines were added by compinstall
zstyle :compinstall filename '~/.zshrc'

autoload -Uz compinit
compinit
# End of lines added by compinstall

bindkey "^[[2~" yank
bindkey "^[[3~" delete-char
bindkey "^[[5~" history-beginning-search-backward
bindkey "^[[6~" history-beginning-search-forward
bindkey "^[[7~" beginning-of-line
bindkey "^[[8~" end-of-line
bindkey "^[[A"  up-line-or-history
bindkey "^[[B"  down-line-or-history
bindkey "^[[H"  beginning-of-line
bindkey "^[[F"  end-of-line
bindkey "^[e"   expand-cmd-path
bindkey "^[[1~" beginning-of-line                      # Pos1
bindkey "^[[4~" end-of-line
bindkey " "     magic-space
bindkey "^[u"   undo
bindkey "^[r"   redo

# a wee bitty bit of emacs
bindkey "^A"    beginning-of-line
bindkey "^E"    end-of-line

bindkey "^R"    history-incremental-search-backward

alias ls='ls -h --color=auto --group-directories-first'
alias df='df -m'
alias lsl='ls -hl --color=auto --group-directories-first'

alias mv='nocorrect mv -i'
alias cp='nocorrect cp -Ri'
alias rm='nocorrect rm -rI'
alias rmf='nocorrect rm -f'
alias rmrf='nocorrect rm -fR'
alias mkdir='nocorrect mkdir'

# Convenient ones
alias tmux="tmux -u attach || tmux -u"
alias wget="wget --continue --content-disposition"
alias grep="grep --colour"
alias feht="feh -FqtV --sort filename"
alias feh="feh -FYqV --sort filename"
alias qmv="qmv --format=do"
alias qcp="qcp --format=do"

alias dvdplay="mplayer2 dvd://1 -dvd-device"

# alias for eix'ing in in separate cache for remotes
alias eixr='eix --cache-file /var/cache/eix/remote.eix'
# (update eix-remote cache via cron)
# @daily /bin/bash -c 'EIX_CACHEFILE="/var/cache/eix/remote.eix" \
#        eix-remote update &> /dev/null'


## Shell functions

# `find` things easily
findhere() { find . -iname "*$1*" }

# Watch YouTube video with Mplayer
youmplayer () { mplayer2 `youtube-dl -g $1` }

# Coloured and lessed diff
udiff() {
        diff -u $1 $2 | pygmentize -g -O encoding=latin1 | less -FRXe
}

# Grep current kernel config for options
krngrep() { zgrep --colour --ignore-case $1 /proc/config.gz }

# Open package homepage
urlix () {
    for url in `eix -e --pure-packages $1 --format '<homepage> '`; do
        $BROWSER "$url" > /dev/null;
    done
}

# Open package's ebuild in editor
ebldopen () { $EDITOR `equery which $1` }

# Open package changelog
ebldlog () {
    # I'll enforce vimpager
    local PAGER="vimpager"
    equery changes -f $1 | $PAGER - -c 'set ft=gentoo-changelog'
}

compdef "_gentoo_packages available" urlix ebldopen ebldlog

# Find and symlink package from overlay
lnspkg () {
    local pkg len LCLPORT LMNDIR
    LMNDIR="/var/lib/layman"
    LCLPORT="/usr/local/portage"
    pkg=($LMNDIR/*/*/$1(N))
    len=${#pkg}
    if [ "$len" -eq 0 ]; then
        echo "No package \"$1\" found in $LMNDIR" >&2
        return 1
    fi
    if [ "$len" -eq 1 ]; then
        pkgpath=(${(s:/:)pkg})
        cat=$pkgpath[5]
        echo "Creating $LCLPORT/$cat..."
        mkdir -p "$LCLPORT/$cat"
        if [ -x "$LCLPORT/$cat/$1" ]; then
            echo "$LCLPORT/$cat/$1 already exists" >&2
            return 1
        fi
        echo "Creating symlink: $LCLPORT/$cat/$1 -> $pkg"
        ln -s "$pkg" "$LCLPORT/$cat/$1"
    else
        echo "Multiple results for \"$1\" in $LMNDIR:" >&2
        echo "$pkg" >&2
        return 1
    fi
}

# Digest and compile latest non-live ebuild in ./
digest_compile_latest() {
    local ebuilds latest_ebuild
    ebuilds=( ./*-^(9999).ebuild )
    if [ ${#ebuilds} -eq 0 ]; then
        echo "No non-live ebuilds in current dir" >&2
        return 1
    fi
    latest_ebuild=$ebuilds[-1]
    ebuild $latest_ebuild digest clean install
}

toggle_history() {
    if [[ -n $HISTFILE ]]; then
        unset HISTFILE
    else
        HISTFILE=~/.histfile
    fi
}

# Convert and read rST document in browser
rstread() {
    local htmlfile
    htmlfile="/tmp/$(basename $1).html"
    rst2html.py $1 $htmlfile
    echo file://$htmlfile
}

# Sync tasks
task-sync () {
    local taskpath="$HOME/dev/taskwarrior/"
    hg fetch -R $taskpath
    task merge $taskpath/data/
    hg commit -m "Autocommit by task-sync on $(hostname)" -R $taskpath && \
        hg push -R $taskpath
}

# Auto-completion from `cmd --help`
compdef _gnu_generic ag

# Complete pumount like umount
compdef _mount pumount

# Completion for media players
compdef _mplayer mplayer2

# Rubyless omploading
ompload() {
    curl -F file1=@"$1" http://ompldr.org/upload | \
        awk '/Info:|File:|Thumbnail:|BBCode:/{gsub(/<[^<]*?\/?>/,"");$1=$1;sub(/^/,"\033[0;    34m");sub(/:/,"\033[0m: ");print}'
}

# Upload to MyOpera
# generate cookie with
# % curl -sLc ~/.config/myoperacookie -d "user=$USER&passwd=$PWD&remember=1" \
#     https://my.opera.com/community/login/index.pl -o /dev/null
mopload() {
    local filename encoded
    filename=$(basename "$1")
    curl -sLb ~/.config/myoperacookie \
         http://upload.my.opera.com/Sterkrig/files/addpic.pl \
         -F file=@"${1}" -F dir="${2}" | grep "$filename" | grep scanfilelink \
            | sed 's:<a href="\(.\+\)" id="scanfilelink".\+$:\1:g'
}

# Notify at
notify_at() { echo sw-notify-send "$2" "$3" | at "$1" }

# use $EDITOR to run Lua code in awesome
evawesome () {
  local EVALFILE
  EVALFILE="$HOME/.cache/awesome/eval.lua"
  $EDITOR "$EVALFILE" && cat "$EVALFILE" | awesome-client
}

# prompt
autoload -U promptinit
promptinit

setopt prompt_subst

prompt_gentoovcs_help () {
  cat <<'EOF'
Standard Gentoo prompt theme with VCS info. Like original, it's color-scheme-able:

  prompt gentoovcs [<promptcolour> [<usercolour> [<rootcolour> [<vcsinfocolour>] \
      [<jobnumcolor> [<histmarkcolour>]]]]]

EOF

}

prompt_gentoovcs_setup () {
  prompt_gentoo_prompt=${1:-'blue'}
  prompt_gentoo_user=${2:-'green'}
  prompt_gentoo_root=${3:-'red'}
  prompt_gentoo_vcs=${4:-'white'}
  prompt_gentoo_job=${5:-'magenta'}
  prompt_gentoo_histoff=${6:-'white'}

  jobs="%F{$prompt_gentoo_job}%(1j. [%j] .)%f"

  if [ "$USER" = 'root' ]
  then
    base_prompt="%B%F{$prompt_gentoo_root}%m%k "
  else
    base_prompt="%B%F{$prompt_gentoo_user}%n@%m%k "
  fi

  base_prompt="%S%F{$prompt_gentoo_histoff}%v%s%f${base_prompt}"

  path_prompt="%B%F{$prompt_gentoo_prompt}%1~"
  vcs_prompt='%B%F{$prompt_gentoo_vcs}${vcs_info_msg_0_:+${vcs_info_msg_0_} }'
  post_prompt="%b%f%k"

  PS1="${jobs}${base_prompt}${vcs_prompt}${path_prompt} %(0?.%#.%S%#%s) $post_prompt"
  PS2="$base_prompt$path_prompt %_> $post_prompt"
  PS3="$base_prompt$path_prompt ?# $post_prompt"

  # vcs_info options
  autoload -Uz vcs_info
  zstyle ':vcs_info:*' enable git svn hg
  zstyle ':vcs_info:*' check-for-changes true
  zstyle ':vcs_info:*' unstagedstr '%F{blue}'
  zstyle ':vcs_info:hg:*' get-revision true
  zstyle ':vcs_info:hg:*' get-mq true
  zstyle ':vcs_info:git:*' formats '[±:%u%b%f]'
  zstyle ':vcs_info:hg:*' formats '[☿:%u%b%f%m]'
  zstyle ':vcs_info:hg:*' actionformats '[☿:%u%b%f-%a]'
  zstyle ':vcs_info:hg:*' branchformat '%b'
  zstyle ':vcs_info:hg:*' patch-format '+%p'
  zstyle ':vcs_info:hg:*' nopatch-format ''

  add-zsh-hook precmd vcs_info
  add-zsh-hook precmd (){
    if [[ -z $HISTFILE ]]; then
      psvar[1]=" H "
    else
      psvar[1]=()
    fi
    echo -ne '\a'
  }
}

if [[ $TERM == "screen" ]] then
  prompt clint
else
  prompt_gentoovcs_setup "$@"
fi

# Zmv!
autoload -U zmv
# Zcalc!
autoload -U zcalc

# Cache completion
zstyle ':completion::complete:*' use-cache 1

# List completions
zmodload zsh/complist
zstyle ':completion:*' menu select
zstyle ':completion:*:default' list-colors ${(s.:.)LS_COLORS}

# Kill processes with completion
zstyle ':completion:*:processes' command 'ps -xuf'
zstyle ':completion:*:processes' sort false
zstyle ':completion:*:processes-names' command 'ps xho command'
zstyle '*' hosts $hosts

# Force rehashing
_force_rehash() {
  (( CURRENT == 1 )) && rehash
  return 1
}

# Load forced rehash
zstyle ':completion:*' completer _oldlist _expand _force_rehash _complete

# Load local file
if [[ -f ~/.zshlocal ]]; then
  source ~/.zshlocal
fi

# vim: softtabstop=2:shiftwidth=2

