# De novo assembly with Colora

#BGA24/sessions #pipeline #Snakemake #assembly #GitPod

This session is part of [**Biodiversity Genomics Academy 2024**](https://thebgacademy.org)

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/thebgacademy/colora) 

## Session Leader(s)

University of Sassari (Sardinia, Italy)

- [Lia Obinu, PhD Student](https://www.linkedin.com/in/lia-obinu/)

## Description

De novo assembly is widely used in biodiversity studies, but understanding all the steps and tools required for a good assembly can be challenging and time-consuming. To address this, we developed Colora, a Snakemake workflow that automates the process to aid scientists in producing complete, chromosome-scale de novo genome assemblies. Colora was primarily developed for plants, but it can be used for any organism and is designed to be user-friendly and versatile. With Colora, anyone can assemble a genome!

## Prerequisites

1. Familiarity with linux command line basics (cd, mv, rm)
2. Knowledge of the Nano editor will be helpful

!!! warning "Please make sure you MEET THE PREREQUISITES and READ THE DESCRIPTION above"

    You will get the most out of this session if you meet the prerequisites above.

    Please also read the description carefully to see if this session is relevant to you.
    
    If you don't meet the prerequisites or change your mind based on the description or are no longer available at the session time, please email damon at thebgacademy.org to cancel your slot so that someone else on the waitlist might attend.

## Useful links

- [Colora GitHub repo](https://github.com/LiaOb21/colora)
- [Examples of workflows](https://github.com/LiaOb21/colora/wiki/Tutorials)
- [Preprint](https://www.biorxiv.org/content/10.1101/2024.09.10.612003v1)
- [Presentation](https://www.canva.com/design/DAGPH2pFcsI/b1NqMIkotTTSeaTZIklk3Q/view?utm_content=DAGPH2pFcsI&utm_campaign=designshare&utm_medium=link&utm_source=editor)

## Colora tutorial on GitPod :snake:

The GitPod is already set-up to activate a Snakemake envirnment and download all the input files we need to run the tutorial. However, some of them are not placed in the right directory automatically. For the success of the workflow it is essential that all the required inputs are placed in the right place, Snakemake won't recognise them otherwise. This means that the location and names of the directories must be exactly the same as those showed here when you run Colora, unless stated otherwise. 

Note: we must use the MAIN terminal on the GitPod.

First of all, let's move into the `colora_pipeline` directory and check out what's inside. When you download Colora from GitHub by yourself, this directory is simply called `colora`.

```
cd colora-pipeline
ls
```

We will run Colora using the data contained in the `test_data` directory. Let's move in there and check what we have:

```
cd test_data 
ls
```

Note: in a regular Colora run (with real data) the `test_data` directory must be named `resources`.

At the moment, we can see only three directories here, which are `raw_hic`, `raw_hifi`, and `raw_ont`. As you may suspect, they contain the raw reads that will be used to run the workflow. 

The test-dataset includes a subset of reads obtained from the organism *Saccharomyces cerevisiae* (a magical yeast :beer: :wine_glass: ). The data come from two different BioProjects:

- HiFi and ONT reads come from the BioProject PRJNA1075684 (strain SPSC01)
- Hi-C reads come from the BioProject PRJNA1013711 (strain YBP2)

This dataset is not supposed to have biological meaning, it has been crated only with the purpose of testing the workflow functionality. To make this test dataset available on GitHub for anyone who downloads the repo, we had to split the read files (for HiFi and ONT), and now we need to join them back together.

Note: this step is not necessary with real data.

```
cd /workspace/colora-pipeline/test_data/raw_hifi

cat hifi_test_SPSC01_SRR27947616_PRJNA1075684aa.fastq.gz hifi_test_SPSC01_SRR27947616_PRJNA1075684ab.fastq.gz > hifi_test_SPSC01_SRR27947616_PRJNA1075684.fastq.gz

rm hifi_test_SPSC01_SRR27947616_PRJNA1075684a*

cd /workspace/colora-pipeline/test_data/raw_ont 

cat ont_test_SPSC01_SRR27947616_PRJNA1075684aa.fastq.gz ont_test_SPSC01_SRR27947616_PRJNA1075684ab.fastq.gz ont_test_SPSC01_SRR27947616_PRJNA1075684ac.fastq.gz > ont_test_SPSC01_SRR27947616_PRJNA1075684.fastq.gz

rm ont_test_SPSC01_SRR27947616_PRJNA1075684a*
```


But reads are not enough - we need some more inputs!

1. Oatk database (https://github.com/c-zhou/OatkDB)

This input is necessary to run [`oatk`](https://github.com/c-zhou/oatk?tab=readme-ov-file), the organelle assembler used by Colora. TGitPod has automatically downloaded the `OatkDB` repo for us, but we need to create a symlink to the necessary files so that Snakemake can access them from the correct location.


```
cd /workspace/colora-pipeline/test_data/
mkdir oatkDB
cd oatkDB

ln -s /workspace/OatkDB/v20230921/dikarya_mito.fam
ln -s /workspace/OatkDB/v20230921/dikarya_mito.fam.h3f
ln -s /workspace/OatkDB/v20230921/dikarya_mito.fam.h3i
ln -s /workspace/OatkDB/v20230921/dikarya_mito.fam.h3m
ln -s /workspace/OatkDB/v20230921/dikarya_mito.fam.h3p
```

2. BUSCO database

This database will be used to perform the quality inspection of the assembly at several stages along the workflow. Even in this case, GitPod downloaded it, but we must be sure it is placed in the right directory to be read by Snakemake.

```
cd /workspace/colora-pipeline/test_data/

mkdir busco_db
cd busco_db

ln -s /workspace/busco_db/saccharomycetes_odb10
```

3. NCBI FCS-GX database

This database is used by fcs-gx to decontaminate the assembly, i.e. to remove contaminants if there are any. For now, we'll use the test database instead of the full one. The full fcs-gx database is massive, so this step in the pipeline is optional. If you don't have enough resources to store the full database or run the decontamination step, you can skip it and try a different decontamination tool after Colora is finished. GitPod has already downloaded the database, but we need to make sure Snakemake can access it from the correct location.

Note: When working with real data, this directory should be named `gxdb`.

```
cd /workspace/colora-pipeline/test_data/

ln -s /workspace/gx_test_db
```

Alright, we're good to go!

Just remember that Snakemake treats the current directory as the home directory. So, we need to head back to the `colora_pipeline` directory to launch Colora :snake: .

```
cd /workspace/colora-pipeline/
snakemake --configfile config/config_test.yaml --software-deployment-method conda --snakefile workflow/Snakefile --cores 8
```
