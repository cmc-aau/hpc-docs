# Submitting jobs (SLURM)
when logging in, you have xxx ressources, dont run anything
use screen or tmux

how to test/development with few ressources
- list of partitions (general, GPU)
- sbatch scripts and examples, cheat sheets
https://hpc-aub-users-guide.readthedocs.io/en/latest/octopus/jobs.html

In order to use a GPU for the deep learning job (or other jobs that require GPU usage), the following flag must be specified in the job script:
#SBATCH --gres=gpu

R SLURM