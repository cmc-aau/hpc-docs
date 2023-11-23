# Conda environments
[Conda](https://docs.conda.io/projects/conda/en/latest/) is an open-source package management system and environment management system that runs on Windows, macOS, and Linux. Conda quickly installs, runs, and updates packages and their dependencies. Conda easily creates, saves, loads, and switches between environments on your local computer. It was created for Python programs but it can package and distribute software for any language. Conda also doesn't require elevated privileges allowing users to install anything with ease. In comparison to [containers](containers.md), Conda is a dependency manager at the Python package level, while containers also manage operating system dependencies at the base OS image level. Containers and conda environments are often used together to ensure complete reproducibility and portability.

[Cheatsheet here](https://docs.conda.io/projects/conda/en/latest/_downloads/843d9e0198f2a193a3484886fa28163c/conda-cheatsheet.pdf)

## Creating and activating an environment
`conda` is notoriously slow for resolving dependencies and creating environments, so it's recommended to use [`mamba`](https://mamba.readthedocs.io/en/latest/) instead of `conda` when creating and manipulating environments as it is MUCH faster (written in C++). `conda` and `mamba` have identical sub-commands and options and can be used interchangibly.

The best practice is to note everything down in a YAML file before you forget things and keep it in the root of the project folder, for example:

**requirements.yml**
```
name: myproject
channels:
  - bioconda
dependencies:
 - minimap2=2.26
 - samtools=1.18
```

Then create the environment with `mamba env create -f requirements.yml`. You can also export an **activated** environment and dump the exact versions used into a YAML file with `mamba env export > requirements.yml`.

Activate and deactivate environments with
```
$ conda activate myproject
$ conda deactivate
```

List available environments with
```
conda env list
```

## Installing packages using pip within conda environments
Software that can only be installed with pip have to be installed in a Conda environment by using pip inside the environment. While issues can arise, per the [Conda guide for using pip in a Conda environment](https://www.anaconda.com/blog/using-pip-in-a-conda-environment), there are some best practices to follow to reduce their likelihood, namely:

 - Use pip only after conda package installs
 - Use conda environments for isolation (Don't perform pip installs in the "root" environment)
 - Recreate the entire environment if changes are needed after pip packages have been installed

An install command would look like:

```
$ python3 -m pip install <package> --no-cache-dir
```

## R and installing R packages within conda environments
Use `r-{package}`, list [here](https://anaconda.org/r/repo?sort=_name&sort_order=asc). Otherwise using [renv](https://rstudio.github.io/renv/articles/renv.html) is **highly recommended** for project reproducibility and portability.
