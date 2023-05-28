bash_profile
```bash
export PATH=/usr/local/bin:$PATH
export EDITOR=vi
```
inputrc
```bash
set editing-mode vi 
#set editing-mode emacs 
set show-all-if-ambiguous on 
set completion-ignore-case on 
set meta-flag on 
set convert-meta off 
set output-meta on 
set bell-style visible 

"\C-l": clear-screen 
"\C-n": next-history 
"\C-p": previous-history 
"\C-a": beginning-of-line 
"\C-e": end-of-line 
"\C-f": forward-char 
"\C-b": backward-char
```
