---
layout: page
title: Reproducibility standards
permalink: /reproducibility/
---

# How to write a reproducible paper
Containers help ensure computational reproducibility, but it can be all for nothing if there are issues with data provenance. Datalad uses git (annex) to version control data and provides tools to monitor the changes processing pipelines make.

Datalad can be install with `conda create -c conda-forge -n datalad datalad git-annex`, then enabled with `source activate datalad`. To create a new datalad repository, run `datalad create -c yoda repository_name`. `-c yoda` is (an optional but highly recommend) datalad procedure that initializes your repository with a standardized file structure.

To version control data, run `datalad save -r -m "commit message"`. This will move your files into the "git annex" and replace them with symlinks from the annex. This keeps your data safe so if you accidentally `rm something`, you can restore it with `git checkout -- something`. Using `datalad save` is useful when setting a project up or making manual hotfixes, but most of the time files will be created or modified by scripts you write. Although it is possible to run your scripts then save the results with `datalad save` afterwords, running your code through the `datalad run` wrapper is preferred because it automatically checks to make sure inputs are available, unlocks and saves outputs, as well as commit the command line used to do the processing. For example, instead of
```
datalad unlock my_inputs
./my_script.sh my_inputs my_outputs
datalad save -r -m "ran my script" my_outputs
```
do
```
datalad run -m "ran my script" -i my_inputs -o my_outputs ./my_script.sh "{inputs}" "{outputs}"
```
Once you've run your analysis with datalad run, make sure your results are reproducible by running `datalad rerun`. Code can be unreproducible in very non-obvious ways (for example, the default way of setting a seed won't work in R if your code is multithreaded), so running twice is the only way to make sure your analysis is reproducible. 

For more information, the [datalad handbook](http://handbook.datalad.org/en/latest/index.html) is an excellent resource.
