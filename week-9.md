---
tags: ggg, ggg2026, ggg201b
---

# Week 9 - Assembly collapse; and FastQC/MultiQC/trimming! - GGG 201B lab section: Genomics (Winter 2026)

3/6/26

Titus Brown, ctbrown@ucdavis.edu

## At the beginning

* start zoom / recording!
* CTB to clean up directory & conda envs

## Examining the results of HW #2 - where does assembly collapse?

We're going to use Python & a Jupyter Notebook to look at this.

### Start a JupyterLab session

Go to [on demand / JupyterLab](https://ondemand.farm.hpc.ucdavis.edu/pun/sys/dashboard/batch_connect/sys/jupyter/session_contexts/new) and start a new session:

* partition: `low`
* cores: 2
* amount of memory: 10
* number of hours: 2

and leave everything else as-is.

Connect to JupyterLab. Scroll down to the bottom and select 'Terminal'.

(This is an alternate Web interface to the HPC than RStudio, but offers similar functionality.)

### Clone the repo

Clone [the 2026-ggg-201b-assembly-collapse repo](https://github.com/ngs-docs/2026-ggg-201b-assembly-collapse) into your account:

```
cd ~/
git clone https://github.com/ngs-docs/2026-ggg-201b-assembly-collapse.git ~/201b-collapse
cd ~/201b-collapse
```

Install the necessary software from [the environment file](https://github.com/ngs-docs/2026-ggg-201b-assembly-collapse/blob/main/environment.yml):

```
module load conda
conda env create \
    -p ~ctbrown/scratch3/2026-conda/$USER/assembly-viz \
    -f environment.yml
```

Now, activate the environment:
```
ln -s ~ctbrown/scratch3/2026-conda/$USER/assembly-viz \
    ~/.conda/envs/
conda activate assembly-viz
```

...and link the software to JupyterLab:
```
python -m ipykernel install --user \
    --name assembly_collapse
```

You can now use this from JupyterLab without activating any environments.

### Open the `assembly-collapse.ipynb` notebook

Use the file navigation in JupyterLab to find and open `assembly-collapse.ipynb` from within the `201b-collapse` location.

It will ask you to select a kernel - select `assembly_collapse`. You may need to reload.

Use the play button to run individual cells in notebook, and the fast forward to re-start and run everything.

### Questions

Three big questions:

* what's going on with all the bad estimates??
* what's going on with the points with  really high coverage? what's the theoretical max coverage of this data set?
* at what coverage value do n50, bp, n_contigs shift in "behavior"? what are "good" values for coverage for each of these? and why are they different?

## Switch to RStudio

OK, let's switch back to RStudio - it has a nicer HTML visualization interface for what we're going to do next. :)

To do this, close your JupyterLab tab, and cancel your JupyterLab job.

Now start a new RStudio:

Now select RStudio, and fill in the following values: (**note: 10 GB, 8 cores**)

Account: ctbrowngrp
Partition: low
Number of cores: 2
Amount of memory: 5
Conda environment: r-4.4.2
Number of hours: 2

leave everything else as-is, and click "Launch".

## Install FastQC, MultiQC, and Trimmomatic

Start up the Terminal, and run:

```
module load conda
```

Then install:
```
conda clean --all -y
conda create -p ~ctbrown/scratch3/2026-conda/$USER/trim \
    -y fastqc multiqc trimmomatic
```

Link, activate:
```
ln -s ~ctbrown/scratch3/2026-conda/$USER/trim \
    ~/.conda/envs/
conda activate trim
```

Grab some RNAseq data from yeast:
```
mkdir -p ~/201b-trim
cd ~/201b-trim
ln -s ~ctbrown/data/ggg-rnaseq/*.fastq.gz .
```

Run FASTQC on these -
```
fastqc *.fastq.gz
```

let's look at one fastqc report - it's in the .html files in the directory.

Now run multiqc to summarize all of them in one report:

```
multiqc .
```

and look at `multiqc_report.html` in the Web browser. What do we see?

### Trim with trimmoatic.

Using [trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic):

Run three increasingly stringent trimmings; what does the `SLIDINGWINDOW:4:15` do?

Least:
```
# least stringent
trimmomatic SE ERR458493.fastq.gz ERR458493_trim.fastq.gz LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36

# more stringent
trimmomatic SE ERR458493.fastq.gz ERR458493_trim2.fastq.gz LEADING:3 TRAILING:3 SLIDINGWINDOW:4:20 MINLEN:36

# most stringent:
trimmomatic SE ERR458493.fastq.gz ERR458493_trim3.fastq.gz LEADING:3 TRAILING:3 SLIDINGWINDOW:4:30 MINLEN:36
```

Summarize with multiqc again:
```
multiqc . --force
```

What's going on?

Let's look at what phred score means... [wikipedia link](https://en.wikipedia.org/wiki/Phred_quality_score)


