## Description

This file is to show you how to analyse metagenomes on ISAAC Open Legacy using part of the [metaWRAP](https://github.com/bxlab/metaWRAP/blob/master/Usage_tutorial.md) pipeline as it has a very detailed guide for you to look into. 

**NOTE: The scheduler and workload manager on ISAAC Open Legacy cluster will be replaced with SLURM on June 30, 2022. You will have to submit SLURM based jobs from July 2022. You may follow instructions in ISAAC_Next_Gen_usage.md to create and submit SLURM jobs.**

## 1. Log into ISAAC Open Legacy
```console
ssh your_username@acf-login.acf.tennessee.edu

# go to scratch directory for the analysis
cd $SCRATCHDIR

# remember the path here, we will need it later. The path should be: /lustre/haven/user/your_username
pwd
```

## 2. Software installation

### 2.1 Install miniconda3

We will use conda to install most of the softwares. The advantage of using conda to install softwares is that the dependecies needed for the software will also be installed, so you do not need to download them separately. Let's install conda first. 

```console
# download miniconda3
 wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

# install miniconda3. You just need to follow the instruction on the screen when you run the command below. 
# Steps during the installation: 
# 1. press ENTER to continue; 
# 2. press q to quit reading terms; 
# 3. answer yes to the question: Do you accept the license terms? [yes|no]; 
# 4. copy the path you got above and edit it for the location: '/lustre/haven/user/your_username/miniconda3'. DO NOT USE THE DEFAULT LOCATION. 
# 5. answer yes to the question: Do you wish the installer to initialize Miniconda3 by running conda init? [yes|no]?
 bash Miniconda3-latest-Linux-x86_64.sh

# now you have installed conda. You can use the following to check the path to conda. You should get output like this:/lustre/haven/user/your_username/miniconda3/condabin/conda
 which conda

# add channels
 conda config --add channels defaults
 conda config --add channels bioconda
 conda config --add channels conda-forge
 conda config --add channels ursky
```

### 2.2 Install [metaWRAP](https://github.com/bxlab/metaWRAP/blob/master/installation/metawrap_installation.md) using conda
```console
# install mamba (replaces conda, but much faster):
 conda install -y mamba 
 
# install metawrap: We will create an conda virtual environment called metawrap-env for metawrap installation. 
 mamba create -y --name metawrap-env --channel ursky metawrap-mg=1.3.2
 conda activate metawrap-env

# To fix the CONCOCT endless warning messages in metaWRAP=1.2+, run
 conda install -y blas=2.5=mkl

# download checkm database. checkM database is to check quality of metagenome-assembled genome
mkdir database
mkdir database/checkm_database

# Now manually download the database:
cd database/checkm_database
wget https://data.ace.uq.edu.au/public/CheckM_databases/checkm_data_2015_01_16.tar.gz
tar -xvf *.tar.gz
rm *.gz
cd ../../

# Now tell CheckM where to find this data 
checkm data setRoot     # CheckM will prompt to to chose your storage location: can use database/checkm_database as the location
# If the above did not work, you can try:
checkm data setRoot database/checkm_database

# NOTE: if you do other metawrap steps that are not included here, you need to download additional databases, such as kraken database for taxonomic annotation.  

# deactivate metawrap-env environment
 conda deactivate
```
### 2.3 Install [GTDB-Tk](https://ecogenomics.github.io/GTDBTk/) for taxonomic annotation
```console
conda create -n gtdbtk -c conda-forge -c bioconda gtdbtk
conda activate gtdbtk

# download the reference data automatically
download-db.sh 

# if it takes too long to download the refernce data using the above command, you can copy the data files from ISAAC.
cp /sw/cs400_centos7.3_acfsoftware/gtdbtk/1.5.0/centos7.6_binary/data/gtdbtk_data.tar.gz database
tar -zxvf database/gtdbtk_data.tar.gz 
```
## 3. Metagenome analysis 

The analysis here is using part of the metaWRAP pipeline, including quality control, low-quality reads removal, assembly and binning. You can find the detailed tutorial for metaWRAP here: https://github.com/bxlab/metaWRAP/blob/master/Usage_tutorial.md

```console
# we will create a directory called metagenome_workshop, and everything we do next will be in this directory
mkdir metagenome_workshop
cd metagenome_workshop

# you can always use 'pwd' to check the path
```
### 3.1 download sequencing data

```console
mkdir RAW_READS
cd RAW_READS

# Sample 1
wget ftp.sra.ebi.ac.uk/vol1/fastq/ERR011/ERR011347/ERR011347_1.fastq.gz
wget ftp.sra.ebi.ac.uk/vol1/fastq/ERR011/ERR011347/ERR011347_2.fastq.gz

# Sample 2
wget ftp.sra.ebi.ac.uk/vol1/fastq/ERR011/ERR011348/ERR011348_1.fastq.gz
wget ftp.sra.ebi.ac.uk/vol1/fastq/ERR011/ERR011348/ERR011348_2.fastq.gz

# Unzip the data
gunzip *.gz

# Back to metagenome_workshop directory
cd ..
```
### 3.2 quality control and reads trimming

metaWRAP read_qc command does three things: 1. check quality of the raw reads using [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/); 2. remove adapters and low-quality reads using [Cutadapt](https://cutadapt.readthedocs.io/en/stable/); 3. check quality of trimmed reads/clean reads using FastQC. You can always run these steps separately using the corresponding software. 

***Note: In order to use metaWRAP, the read files must end with _1.fastq and _2.fastq***

**Replace the email address with your own, so you can get emails about your job when it begins, ends and aborts due to abnormal execution**

First use emacs to create script file:
```console
emacs read_qc.sh
```
Now copy and paste the following, remember to change email address. We will use the bigmem partition for most of the jobs here. The bigmem partition has a bigger memory compare to the other partitions on ISAAC, but it only has 4 nodes, 48 cores per node, 1,536 GB memory per node. You can use bigmem for jobs that requires large memory usage and use other partitions for smaller jobs. ***NOTE: you can only run jobs for maximum 24 hours when using bigmem partition. The long-utk qos does not work for this partition***
```
#!/bin/sh
#PBS -S /bin/bash
#PBS -A ACF-UTK0011
#PBS -l partition=bigmem
#PBS -l nodes=1:ppn=8,walltime=4:00:00
#PBS -m abe 
#PBS -M your_email_address
cd $PBS_O_WORKDIR
conda activate metawrap-env # we need to activate metawrap-env to use metawrap commands
mkdir READ_QC # create a directory READ_QC for the output
for s in RAW_READS/ERR*_1.fastq; do 
metawrap read_qc -1 $s -2 ${s/_1.fastq/_2.fastq} -t 32 --skip-bmtagger -o READ_QC/$(basename $s _1.fastq)
done 
```
Save the file by pressing control + x + s, and then control + z to quit. you will do the same for the following scripts editing.

We use `for` loop above to run the read_qc command. The `for` loop means the read_qc command will run for each sequence file within the RAW_READS directory. '-1 $s': indicate the forward sequences; '-2 ${s/_1.fastq/_2.fastq}': replace the file extension _1.fastq with _2.fastq for the reverse sequences; '$(basename $s _1.fastq)': only want to use the names before _1.fastq as output folder. If you do not know what this replacement means, you can alwasy use `echo` to print it out and see what has been changed.
```console
for s in RAW_READS/ERR*_1.fastq; do 
echo $s
echo ${s/_1.fastq/_2.fastq}
echo $(basename $s _1.fastq)
done
```
If you cannot follow the for loop at this point, you can run the command for each sample like the followings. But this will be time consuming if you have many samples.
```console
emacs read_qc.sh
```
```
#!/bin/sh
#PBS -S /bin/bash
#PBS -A ACF-UTK0011
#PBS -l partition=bigmem
#PBS -l nodes=1:ppn=8,walltime=4:00:00
#PBS -m abe 
#PBS -M your_email_address
cd $PBS_O_WORKDIR
conda activate metawrap-env # we need to activate metawrap-env to use metawrap commands
mkdir READ_QC # create a directory READ_QC for the output
metawrap read_qc -1 RAW_READS/ERR011347_1.fastq -2 RAW_READS/ERR011347_2.fastq -t 32 --skip-bmtagger -o READ_QC/ERR011347
metawrap read_qc -1 RAW_READS/ERR011348_1.fastq -2 RAW_READS/ERR011348_2.fastq -t 32 --skip-bmtagger -o READ_QC/ERR011348
```
Submit the job. The job should finish in ~12 min after the job starts with 8 cores of bigmem node
```console
qsub read_qc.sh
```
### 3.3 rename clean reads
You do not have to submit a job to do this step. You can run the commands on your login node as it is a small job, but you can run it as a dependency job this way.
```console
emacs move_rename_clean_reads.sh 
```
```
#!/bin/sh
#PBS -S /bin/bash
#PBS -A ACF-UTK0011
#PBS -l nodes=1:ppn=2,walltime=1:00:00
#PBS -m abe 
#PBS -M your_email_address
cd $PBS_O_WORKDIR
mkdir CLEAN_READS
for i in READ_QC/*; do 
b=${i#*/}
mv ${i}/final_pure_reads_1.fastq CLEAN_READS/${b}_1.fastq
mv ${i}/final_pure_reads_2.fastq CLEAN_READS/${b}_2.fastq
done
```
You can only submit this job after the read_qc job is completed, because you need the output file from read_qc. Another option is that you can submit it as a dependency job, so you can just let your jobs run one after another without keeping an eye on it all the time. You might want to do some debbugging to make sure your scripts are working, otherwise your dependency jobs will be cancelled if the previous job failed.

Option 1: submit move_rename_clean_reads.sh after read_qc is completed
```console
qsub move_rename_clean_reads.sh 
```
Option 2: submit move_rename_clean_reads.sh as a dependency job. The following means that your move_rename_clean_reads job will only be run after the read_qc job finishes without error. Replace jobid_of_read_qc.sh with the real jobid you got.
```console
qusb -W depend=afterok:jobid_of_read_qc.sh move_rename_clean_reads.sh  
```
### 3.4 assembly with megahit

We will co-assembly the two samples using megahit as it is much faster. You can also replace --megahit with --metaspades to use metaspades as the assembler. Metaspades has better performance, but it takes very long for large datasets. You can also run [megahit](https://github.com/voutcn/megahit) or [metaspapdes](https://github.com/ablab/spades) on your own without using metawrap command. Metawrap assembly here does two steps: 1. assembly with `megahit`; 2. contig quality evaluation using `quast`.
```console
emacs assembly_megahit.sh
```
```
#!/bin/sh
#PBS -S /bin/bash
#PBS -A ACF-UTK0011
#PBS -l partition=bigmem
#PBS -l nodes=1:ppn=12,walltime=10:00:00
#PBS -m abe 
#PBS -M your_email_address
cd $PBS_O_WORKDIR
conda activate metawrap-env
cat CLEAN_READS/ERR*_1.fastq > CLEAN_READS/ALL_READS_1.fastq # Concatinate the forward reads from all the samples for co-assembly
cat CLEAN_READS/ERR*_2.fastq > CLEAN_READS/ALL_READS_2.fastq # Concatinate the reverse reads from all the samples for co-assembly
metawrap assembly -1 CLEAN_READS/ALL_READS_1.fastq -2 CLEAN_READS/ALL_READS_2.fastq -m 300 -t 48 --megahit -o ASSEMBLY_megahit 
```
Submit the assembly job after move_rename_clean_reads is completed. The job should finish in ~15 min after job starts with 12 cores of bigmem node.
```console
qsub assembly_megahit.sh
```
Now let's have a look at the contig sequences. You will see that each contig name starts with ">". Press 'q' to quit viewing
```console
less ASSEMBLY_megahit/final_assembly.fasta
```
If we want to get all the contig names, grep is useful for that.the first part 'grep ">" ASSEMBLY_megahit/final_assembly.fasta': extract the lines with ">" from the contig file; the second part '> contig_names.txt': save those lines into a file called contig_names.txt
```console
grep ">" ASSEMBLY_megahit/final_assembly.fasta > contig_names.txt 

# if we want to count how many lines/contigs
grep ">" ASSEMBLY_megahit/final_assembly.fasta | wc -l
```
You can see all the contig names in this file.
```console
less contig_names.txt
```
There are many other possibilities with `grep` command, please do read more in the future.

### 3.5 binning

metawrap binning has several binning softwares for you to choose. You can just use one binning software. Here we will use three binning softwares: [MetaBAT2](https://bitbucket.org/berkeleylab/metabat/src/master/), [MaxBin2](https://sourceforge.net/projects/maxbin2/) and [CONCOCT](https://github.com/BinPro/CONCOCT). Later we will use the bin_refinement command to consolidate bins from the three softwares.

You can also run metabat2, maxbin2 and concoct binning separately without using metawrap. If you do so, first you need to map your clean reads back to the contigs using [bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml), [BBMap](https://jgi.doe.gov/data-and-tools/bbtools/bb-tools-user-guide/bbmap-guide/) or other mapping softwares, then you follow the instructions from binning softwares. metawrap binning here does two steps: 1. binning with the selected softwares; 2. check the quality of recovered bins using [checkM](https://github.com/Ecogenomics/CheckM/wiki).

```console
emacs binning.sh  
```
```
#!/bin/sh
#PBS -S /bin/bash
#PBS -A ACF-UTK0011
#PBS -l partition=bigmem
#PBS -l nodes=1:ppn=8,walltime=10:00:00
#PBS -m abe 
#PBS -M your_email_address
cd $PBS_O_WORKDIR
conda activate metawrap-env
metawrap binning -o INITIAL_BINNING -t 32 -a ASSEMBLY_megahit/final_assembly.fasta --metabat2 --maxbin2 --concoct CLEAN_READS/ERR*fastq --universal
```
Submit after assembly_megahit is completed. ~2.5 hours run
```console
qsub binning.sh 
```
### 3.6 binning refinement

We will select bins with completeness >50% and contamination <10% (-c 50 -x 10).

```console
emacs binning_refinement.sh   
```
```
#!/bin/sh
#PBS -S /bin/bash
#PBS -A ACF-UTK0011
#PBS -l partition=bigmem
#PBS -l nodes=1:ppn=8,walltime=10:00:00
#PBS -m abe 
#PBS -M your_email_address
cd $PBS_O_WORKDIR
conda activate metawrap-env
metawrap bin_refinement -o BIN_REFINEMENT -t 32 -A INITIAL_BINNING/metabat2_bins/ -B INITIAL_BINNING/maxbin2_bins/ -C INITIAL_BINNING/concoct_bins -c 50 -x 10
```
Submit after metawrap_binning is completed. ~1 hour run
```console
qsub binning_refinement.sh
```
### 3.7 bin abundance calculation

```console
emacs bin_abundance.sh    
```
```  
#!/bin/sh
#PBS -S /bin/bash
#PBS -A ACF-UTK0011
#PBS -l partition=bigmem
#PBS -l nodes=1:ppn=8,walltime=4:00:00
#PBS -m abe 
#PBS -M your_email_address
cd $PBS_O_WORKDIR
conda activate metawrap-env
metawrap quant_bins -b BIN_REFINEMENT/metawrap_50_10_bins -o QUANT_BINS -a ASSEMBLY_megahit/final_assembly.fasta CLEAN_READS/ERR*fastq
```          
Submit after binning_refinement is completed. ~5 min run
```console
qsub bin_abundance.sh
```
***We will stop using metawrap commands from here, so we need to deactivate metawrap environment by typing `conda deactivate`***

### 3.8 taxonomic classification [`gtdbtk`](https://ecogenomics.github.io/GTDBTk/) 

We will use gtdbtk to assign taxonomy. gtdbtk uses [GTDB database](https://gtdb.ecogenomic.org/)
```console
emacs gtdbtk.sh 
```
```
#!/bin/sh
#PBS -S /bin/bash
#PBS -A ACF-UTK0011
#PBS -l partition=beacon
#PBS -l nodes=1:ppn=16,walltime=24:00:00
#PBS -m abe 
#PBS -M your_email_address
cd $PBS_O_WORKDIR 
conda activate gtdbtk
# we need to export the GTDBTK_DATA_PATH since the reference data was not downloaded automatically (we copied it from ISAAC's pre-installation.
export GTDBTK_DATA_PATH=/lustre/haven/user/your_username/database/release202 # don't forget to replace 'your_username' with your own. 
gtdbtk classify_wf --cpus 1 --genome_dir BIN_REFINEMENT/metawrap_50_10_bins --out_dir gtdbtk_output --extension fa --pplacer_cpus 1
```
Submit after binning_refinement is completed
```console
qsub gtdbtk.sh 
```
### 3.9 functional annotation using `hmmsearch`

Here we will use `hmmsearch` to annotate bins against PFAM database. First we will use `prodigal` to predict genes for each bin.
```console
emacs prodigal_bins.sh 
```
```
#!/bin/sh
#PBS -S /bin/bash
#PBS -A ACF-UTK0011
#PBS -l partition=bigmem
#PBS -l nodes=1:ppn=4,walltime=2:00:00
#PBS -m abe 
#PBS -M your_email_address
cd $PBS_O_WORKDIR
module load prodigal/2.6.3 # load prodigal 
mkdir prodigal
for s in BIN_REFINEMENT/metawrap_50_10_bins/*.fa;do
prodigal -a prodigal/$(basename $s .fa).aa.fa -d prodigal/$(basename $s .fa).nuc.fa -i $s -o prodigal/$(basename $s .fa).gff -p meta 
done
```
Submit after binning_refinement is completed. ~1 min run
```console
qsub prodigal_bins.sh  
```

Now run hmmsearch on the protein sequences of predicted genes against PFAM database. First we need to download the database and unzip the file.
```console
wget http://ftp.ebi.ac.uk/pub/databases/Pfam/current_release/Pfam-A.hmm.gz 
gunzip Pfam-A.hmm.gz 
```
Search amino acid sequences with HMMER against the Pfam database
```console
emacs hmmsearch_pfam.sh
```
```
#!/bin/sh
#PBS -S /bin/bash
#PBS -A ACF-UTK0011
#PBS -l partition=bigmem
#PBS -l nodes=1:ppn=4,walltime=4:00:00
#PBS -m abe 
#PBS -M your_email_address
cd $PBS_O_WORKDIR
module load hmmer/3.1b2
mkdir pfam
for s in prodigal/*.aa.fa;do
hmmsearch --tblout pfam/$(basename $s .aa.fa)_hmmsearch.txt -E 1e-5 --cpu 8 Pfam-A.hmm $s
done
```
Submit after prodigal_bins is completed. ~15 min run
```console
qsub hmmsearch_pfam.sh 
```
### Bonus: functional annotation using [PROKKA](https://github.com/tseemann/prokka)

The good thing of using PROKKA is that it does both gene prediction and annotation. PROKKA is not designed for metagenomes, but it does work on them. There was an issue that discussed [here](https://github.com/tseemann/prokka/issues/203) when using PROKKA on metagenomes. We will use the prokka that installed as metawrap dependency. 

Annotate bacterial bins.
```console
emacs prokka_bacteria.sh
```
```
#!/bin/sh
#PBS -S /bin/bash
#PBS -A ACF-UTK0011
#PBS -l nodes=1:ppn=8,walltime=4:00:00
#PBS -m abe 
#PBS -M your_email_address
cd $PBS_O_WORKDIR 
conda activate metawrap-env # we need to activate metawrap environment to use PROKKA 
mkdir prokka
for s in BIN_REFINEMENT/metawrap_50_10_bins/bin.{2,3,4,5,6}.fa;do # NOTE here we use contig sequences of each bin, not the protein sequences of genes got from prodigal
prokka --outdir prokka/$(basename $s .fa) --prefix $(basename $s .fa) --metagenome $s --cpus 8 --kingdom Bacteria # kingdom is Bacteria by default
done
```
submit after binning_refinement is completed
```console
qsub prokka_bacteria.sh 
```

We have 1 archaeal bin (bin.1.fa) based on GTDB-Tk results. Annotate archaeal bin. 
```console
emacs prokka_archaea.sh
```
```
#!/bin/sh
#PBS -S /bin/bash
#PBS -A ACF-UTK0011
#PBS -l nodes=1:ppn=8,walltime=4:00:00
#PBS -m abe 
#PBS -M your_email_address
cd $PBS_O_WORKDIR 
conda activate metawrap-env # we need to activate metawrap environment to use PROKKA 
mkdir prokka 
for s in BIN_REFINEMENT/metawrap_50_10_bins/bin.1.fa;do # NOTE here we use contig sequences of each bin, not the protein sequences of genes got from prodigal
prokka --outdir prokka/$(basename $s .fa) --prefix $(basename $s .fa) --metagenome $s --cpus 8 --kingdom Archaea # kingdom changed to Archaea
done
```
submit after binning_refinement is completed
```console
qsub prokka_archaea.sh
```

Alternatively, we can use the metawrap annotate_bins module which also uses PROKKA for functional annotation on the bins.
```console
emacs metawrap_annotate_bins.sh
```
```
#!/bin/sh
#PBS -S /bin/bash
#PBS -A ACF-UTK0011
#PBS -l nodes=1:ppn=8,walltime=5:00:00
#PBS -m abe 
#PBS -M your_email_address
cd $PBS_O_WORKDIR 
conda activate metawrap-env
metaWRAP annotate_bins -o FUNCT_ANNOT -b BIN_REFINEMENT/metawrap_50_10_bins -t 8
```
Submit after binning_refinement is completed
```console
qsub metawrap_annotate_bins.sh
```