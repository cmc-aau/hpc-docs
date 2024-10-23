# Downloading data from NCBI SRA
https://bioinformatics.ccr.cancer.gov/docs/b4b/Module1_Unix_Biowulf/Lesson6/

## Download sra-tools container
```
singularity pull docker://ncbi/sra-tools
```

## Get data
bioproject: PRJNA192924
sra: SRR1154613
prefetch first, then use fasterq-dump
```
singularity run sra-tools_latest.sif prefetch SRR1154613
singularity run sra-tools_latest.sif fasterq-dump --threads $(nproc) --progress --split-3 SRR1154613
```

## Download SRA data from ENA instead
https://www.ebi.ac.uk/ena/browser/view/SRR1154613
