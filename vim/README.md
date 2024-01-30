## delete blank lines

```
:g/^$/d
# or
g/^\s*$/d
```

Brief explanation of :g
```
:[range]g/pattern/cmd
```
This acts on the specified [range] (default whole file), by executing the Ex command cmd for each line matching pattern (an Ex command is one starting with a colon such as :d for delete). Before executing cmd, "." is set to the current line.

##  delete all trailing whitespace from each line

```
:%s/\s\+$//e
```

## Delete Lines Beginning With A Character

E.g. remove all comments: 
```
:g/^\#/d
```

## Replace

```
#case insensitive, ask for confirmation
:%s/foo/bar/gci

#case sensitive, don't ask for confirmatio
:%s/foo/bar/gI
```

## Open several files in one window (e.g. 2 files side b side)

In a shell:
```
vim -o file1.txt file2.txt file3.txt
```
-p[N]  Open N tab pages (default: one for each file)

-o[N]  Open N windows (default: one for each file)

-O[N]  Like -o but split vertically

Inside vim:

```
Ctrl+W, S (case does not matter) for horizontal splitting

Ctrl+W, v (lower case) for vertical splitting

Ctrl+W, q (lower case) to close one

:Ex <directory> , lets you browse for the file from the given directory. Or, use

:Sex (Split & Explore current file's directory)
:Hex (Horizontal Split and Explore)

```

## To set up tabs and cursors:
in ~/.vimrc, add: 
```
autocmd FileType yaml set ts=2 sw=2 et ai
autocmd FileType yaml set cursorcolumn
```

## To copy yaml examples with no multiplying identations
:set paste 