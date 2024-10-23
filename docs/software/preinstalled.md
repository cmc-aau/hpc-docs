# Pre-installed software
In addition to software management tools, there are a few things that are installed natively.

The following software is pre-installed:

 - CLC (on `axomamma` only)

## CLC Genomics Workbench
As described here, you can run graphical apps in a SLURM job while the windows show up on your own computer by using the `--x11` option to `srun` and `salloc`, as described here https://cmc-aau.github.io/biocloud-docs/slurm/jobsubmission/#graphical-apps-gui. For example to run CLC you could login to a login node, then run:
```
srun --cpus-per-task 4 --mem 4G --nodelist axomamma --x11 /usr/local/CLCGenomicsWorkbench24/clcgenomicswb24
```

## ARB
ARB is old and unmaintained. Version 6 is available through conda, but the latest version 7 is not, and the only way to run it on servers running a later Ubuntu version than 20.04 is through a container:
```
srun --cpus-per-task 4 --mem 4G --x11 apptainer run -B ~/.Xauthority -B /projects /home/bio.aau.dk/ksa/projects/biocloud-software/containers/apptainer/arb/arb-7.0.sif
```

## Alphafold
databases