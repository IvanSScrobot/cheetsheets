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