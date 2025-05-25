
# Helpful documentation

https://junegunn.github.io/fzf/


# Fuzzy File Find Aliases

```bash
export FZ_OPEN_OPTS="--walker-skip .git,node_modules,target --preview 'bat -n --color=always {}' --bind 'ctrl-/:change-preview-window(down|hidden|)' --preview-window '~4,+{2}+4/3,<80(up)'"

# Open a file in vim from home directory
alias ff='vim $(fd . '$HOME' | fzf '$FZ_OPEN_OPTS')'

# Open a file in vim from the current directory
alias ffcurr='vim $(fd . | fzf )'

# Open a file in  vs code from current directory 
alias ffc='code $(find . | fzf --multi)'
```


# Fuzzy text search

Following is yoinked from the above docco:

```bash
frg() (
  RELOAD='reload:rg --column --color=always --smart-case {q} || :'
  OPENER='if [[ $FZF_SELECT_COUNT -eq 0 ]]; then
            vim {1} +{2}     # No selection. Open the current line in Vim.
          else
            vim +cw -q {+f}  # Build quickfix list for the selected items.
          fi'
  fzf --disabled --ansi --multi \
      --bind "start:$RELOAD" --bind "change:$RELOAD" \
      --bind "enter:become:$OPENER" \
      --bind "ctrl-o:execute:$OPENER" \
      --bind 'alt-a:select-all,alt-d:deselect-all,ctrl-/:toggle-preview' \
      --delimiter : \
      --preview 'bat --style=full --color=always --highlight-line {2} {1}' \
      --preview-window '~4,+{2}+4/3,<80(up)' \
      --query "$*"
)
```