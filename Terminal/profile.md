
# Bat Alias
```bash
alias bathelp='bat --plain --language=help'
help() {
    "$@" --help 2>&1 | bathelp
}
# use like k9s help
```

# docker exec (fzf)

```
# Select a docker container to start and attach to
dbash() {
  local cid
  cid=$(docker ps | sed 1d | fzf -1 -q "$1" | awk '{print $1}')

  [ -n "$cid" ] && docker exec -it "$cid" bash
}
```
# git commit preview and branch checkout

```shell
fbr() {
  local branches branch
  branches=$(git --no-pager branch -vv) &&
  branch=$(echo "$branches" | fzf +m) &&
  git checkout $(echo "$branch" | awk '{print $1}' | sed "s/.* //")
}

alias glNoGraph='git log --color=always --format="%C(auto)%h%d %s %C(black)%C(bold)%cr% C(auto)%an" "$@"'
_gitLogLineToHash="echo {} | grep -o '[a-f0-9]\{7\}' | head -1"
_viewGitLogLine="$_gitLogLineToHash | xargs -I % sh -c 'git show --color=always % | delta'"

# fshow_preview - git commit browser with previews
fshow() {
    glNoGraph |
        fzf --no-sort --reverse --tiebreak=index --no-multi \
            --ansi --preview="$_viewGitLogLine" \
                --header "enter to view, alt-y to copy hash" \
                --bind "enter:execute:$_viewGitLogLine   | less -R" \
                --bind "alt-y:execute:$_gitLogLineToHash | xclip"
}
```
# jwt func decode

```bash
# just use this: https://github.com/mike-engel/jwt-cli
```

# fzf and rg and bat and all the things

```shell
# put this as 'ff' in your $PATH somewhere, e.g. ~/.local/bin/ff
#!/usr/bin/env bash

# Fuzzy finder script for terminal quick actions

# Check for help flag first
if [[ "$1" == "-h" || "$1" == "--help" ]]; then
    echo "Usage: ff [OPTIONS]"
    echo ""
    echo "Fuzzy finder, helpful for your terminal quick action needs."
    echo ""
    echo "Options:"
    echo "  -f            Fuzzy find with ripgrep over files instead of content (default)"
    echo "  -c            Open selection in vs code instead of vi default"
    echo "  -h, --help    Show this help message"
    echo ""
    echo "Examples:"
    echo "  ff"
    echo "  ff -fc"
    echo "  ff -f"
    exit 0
fi

# Initialize flags
f_flag=false
c_flag=false

# Parse options
while getopts "fc" opt; do
    case $opt in
        f)
            f_flag=true
            ;;
        c)
            c_flag=true
            ;;
        \?)
            echo "Invalid option: -$OPTARG" >&2
            exit 1
            ;;
    esac
done

# Shift past the options
shift $((OPTIND - 1))

# Set RELOAD and OPENER based on flags
if [[ "$f_flag" == true ]]; then
    # Files mode: search filenames only
    RELOAD='reload:rg --files --color=always | rg --color=always {q} || :'
    PREVIEW='bat --style=full --color=always {1}'
else
    # Content mode: search file contents (default)
    RELOAD='reload:rg --column --color=always --smart-case {q} || :'
    PREVIEW='bat --style=full --color=always --highlight-line {2} {1}'
fi

if [[ "$c_flag" == true ]]; then
    OPENER='if [[ $FZF_SELECT_COUNT -eq 0 ]]; then
                code -g {1}:${2:-1}
            else
                code {+f}
            fi'
else   
    OPENER='if [[ $FZF_SELECT_COUNT -eq 0 ]]; then
                vi {1} +{2}
            else
                vi +cw -q {+f}
            fi'
fi  

fzf --disabled --ansi --multi \
    --bind "start:$RELOAD" --bind "change:$RELOAD" \
    --bind "enter:become:$OPENER" \
    --bind "ctrl-o:execute:$OPENER" \
    --bind 'alt-a:select-all,alt-d:deselect-all,ctrl-/:toggle-preview' \
    --delimiter : \
    --preview "$PREVIEW" \
    --preview-window '~4,+{2}+4/3,<80(up)' \
    --query "$*"
```