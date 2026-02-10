---
site: sandpaper::sandpaper_site
---

This lesson teaches how to share Python code, both in the form of
packages, and reproducible scripts. The teaching philosophy is to cover
"good enough" practices, with more content linked out when necessary.

## Prerequisites

-   [ ] Basic understanding of Python syntax
-   [ ] Basic understanding of Shell scripts

::: spoiler
### Do I know enough Python?

It will suffice to know how to use the interactive terminal or a
[Jupyter instance](https://jupyter.org/try-jupyter/lab/)…

``` python
a = [3,5,6.0]

def mysquarer(_a):
    for x in _a:
        x=x**2
    return _a

mysquarer(a)
```

and similar concepts.
:::

::: spoiler
### Do I know enough Shell scripting?

Navigation of the file system through a terminal will suffice, as well
as remembering that **everything is a file** on Unix systems, as far as
the contents of the lesson at any rate.

``` {.bash eval="never"}
# get files in directory
ls
# get directory name
pwd
# view first few lines of a file
head readme.org
```
:::
