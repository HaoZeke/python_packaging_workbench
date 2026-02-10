# Python Dissemination Lesson

A lesson on the dissemination of Python code as scripts and packages,
implemented using the [The Carpentries Workbench][workbench].

To take updates from upstream easily, minimal changes are made here, so setup
instructions follow on the associated [pixi_envs section].

``` sh
# setup locally
gh repo clone HaoZeke/pixi_envs
cd pixi_envs/teaching/python_packaging_workbench
gh repo clone HaoZeke/python_packaging_workbench
pixi s
cd python_packaging_workbench
# serve locally
Rscript -e 'sandpaper::serve()'
```

[workbench]: https://carpentries.github.io/sandpaper-docs/
[pixi_envs section]: https://github.com/HaoZeke/pixi_envs/tree/main/teaching/python_packaging_workbench
