# Conda environments
[Conda](https://docs.conda.io/projects/conda/en/latest/) is an open-source package management system and environment management system that runs on Windows, macOS, and Linux. Conda quickly installs, runs, and updates packages and their dependencies, where a sophisticated dependency solver allows installing multiple tools into the same environment at once without introducing conflicts between required versions of the individual dependencies. This is ideal for scientific projects where reproducibility is key - you can simply create a separate conda environment for each individual project.

Conda was initially created for Python packages but it can package and distribute software for any language. Conda also doesn't require elevated privileges allowing users to install anything with ease. Most tools are already available in the default [Anaconda repository](https://anaconda.cloud/package-categories), but other community-driven channels like [bioconda](https://bioconda.github.io/) allow installing practically anything. In comparison to [containers](containers.md), Conda is a dependency manager at the Python package level, while containers also manage operating system dependencies at the base operating system level, hence containers and conda environments are often used together to ensure complete reproducibility and portability, see for example the [biocontainers.pro](https://biocontainers.pro/) project.

[Cheatsheet here](https://docs.conda.io/projects/conda/en/latest/_downloads/843d9e0198f2a193a3484886fa28163c/conda-cheatsheet.pdf)

## Creating and activating an environment
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

Then create the environment with `conda env create -f requirements.yml`. You can also export an **activated** environment created previously and dump the exact versions used into a YAML file with `conda env export > requirements.yml`.

???+ "Note"
      When you export a conda environment to a file the file may also contain a host-specific `prefix` line, which should be removed.

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
Software that can only be installed with pip have to be installed in a Conda environment by using pip inside the environment. While issues can arise, per the [Conda guide for using pip in a Conda environment](https://www.anaconda.com/blog/using-pip-in-a-conda-environment), there are some best practices to follow to reduce their likelihood:

 - Use pip only after conda package installs
 - Use conda environments for isolation (Don't perform pip installs in the `base` environment)
 - Recreate the entire environment if changes are needed after pip packages have been installed
 - Use `--no-cache-dir` with any `pip install` commands

After activating the conda environment an install command would look like the following:
```
$ python3 -m pip install <package> --no-cache-dir
```

If you then export the conda environment to a YAML file using `conda env export > requirements.yml`, software dependencies installed using pip should show under a separate `- pip:` field, for example:
```
name: myproject
channels:
  - bioconda
dependencies:
 - minimap2=2.26
 - samtools=1.18
 - pip:
   - virtualenv==20.25.0
```

Be aware that specific versions are specified using double `==` with `pip` dependencies.

## R and installing R packages within conda environments
Use the notation `r-{package}` to install R and required R packages within an environment, see the list of packages [here](https://anaconda.org/r/repo?sort=_name&sort_order=asc). Alternatively using [renv](https://rstudio.github.io/renv/articles/renv.html) is **highly recommended** for project reproducibility and portability if you need to install many packages.

## VS Code and conda environments
To ensure VS Code uses for example R and Python installations in conda environments, you can make a file `.vscode/settings.json` in the project folder and write for example:
```shell
{
  "r.rterm.linux": "${userHome}/.conda/envs/myproject/bin/R",
  "python.defaultInterpreterPath": "${userHome}/.conda/envs/myproject/bin/python"
}
```

Read more details [here](https://code.visualstudio.com/docs/python/environments).
