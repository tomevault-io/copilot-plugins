## limbo-fmt

> The Inferno OS shell supports most typical UNIX command line utilities like cat, ls, cd, wc, pwd, uniq, echo. `inferno` should be run in the root folder of the project so that it sees the current working directory mounted on `/n/local`

# compile limbo .b file
```
limbo file.b
```

# build the project
```
mk install
```

# execute a .dis file
The Inferno OS shell supports most typical UNIX command line utilities like cat, ls, cd, wc, pwd, uniq, echo. `inferno` should be run in the root folder of the project so that it sees the current working directory mounted on `/n/local`
```
inferno "ls|wc"
inferno lfmt testfile.b
```


# debug limbo program
inferno debug 
> run /dis/ls.dis
> stack
> next
```

# docs
Get the documentation for module and device interfaces.
```
inferno man sys   # man page for the system calls
inferno man 2 string # docs for string library
```

---
> Source: [caerwynj/limbo-fmt](https://github.com/caerwynj/limbo-fmt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
