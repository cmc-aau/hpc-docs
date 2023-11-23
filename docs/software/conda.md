# Conda environments
Environments on shared storage, use mamba to create environments, much faster
```
$ mamba create -n minimap2 -c bioconda minimap2
```

```
$ conda activate minimap2
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
