---
tags: ggg, ggg2026, ggg201b
---

[toc] 

# Week 10 - trimming; and an RNAseq workflow on yeast - GGG 201B lab section: Genomics (Winter 2026)

3/13/26

## At the beginning

* start zoom / recording!
* HW: graded, all is well; a few comments to send out on assembly
* office hours on Monday: 11-1pm. Zoom and in person in Shields 362.

## Start a new RStudio

Go to https://ondemand.farm.hpc.ucdavis.edu/

Now select RStudio, and fill in the following values:

Account: ctbrowngrp
Partition: low
Number of cores: 4
Amount of memory: 10
Conda environment: r-4.4.2
Number of hours: 2

leave everything else as-is, and click "Launch".

## QC and trimming, cont'd

Continuing from [last week](https://hackmd.io/1NlDbsXzRa2GNGWmu4sQ1Q?view#Install-FastQC-MultiQC-and-Trimmomatic), 

Re-establish yourself
```
module load conda
conda activate trim

cd ~/201b-trim
```

Let's take a quick look at a FastQC report. In particular, let's take a look at the quality score report.

(Brief discussion of Illumina sequence ensues)

### Trim with trimmomatic.

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

What is changing in these things?

Summarize with multiqc again:
```
fastqc *trim*.fastq.gz
multiqc . --force
```

What's going on?

Let's look at what phred score means... [wikipedia link](https://en.wikipedia.org/wiki/Phred_quality_score)

tl;dr
- trimming at higher quality scores is a pretty stringent thing to do and can discard a lot of your data
- unless you're doing inference (e.g. mapping/variant calling) with low coverage, IMO you're better of doing only light trimming and then letting downstream tools deal with errors in their own way.

## A simple RNAseq workflow on yeast

Let's talk about RNAseq!

For the rest of today, we are going to check out a simple RNAseq workflow!

The repository is [here](https://github.com/ngs-docs/2026-ggg-201b-rnaseq).

### Clone the repository

In the Terminal:

```
cd ~/
git clone https://github.com/ngs-docs/2026-ggg-201b-rnaseq 201b-rnaseq
```

## Install the software 

(You only need to do this once; if you've already done it for the homework, don't do it again! You can check by doing `conda activate 2026-rnaseq`; if it works, don't run the next few commands.)

```
conda env create \
    -p ~ctbrown/scratch3/2026-conda/$USER/2026-rnaseq \
    -f ~/201b-rnaseq/environment.yml
```

Link and activate the environment:
```
ln -s ~ctbrown/scratch3/2026-conda/$USER/2026-rnaseq \
    ~/.conda/envs/
```

## Restart RStudio in your 2026-rnaseq environment.

Once the installation succeeds, we need to re-start RStudio to take advantage of the stuff we installed.

Close the RStudio tab, go _back_ to https://ondemand.farm.hpc.ucdavis.edu/, cancel your RStudio Server, and start a new one. This time enter '2026-rnaseq' in the conda environment box.

## Run the command line part of the workflow

In the Terminal,
```
conda activate 2026-rnaseq

cd ~/201b-rnaseq/
snakemake -c 4
```

This will take a few minutes.

What's the output of this workflow?

Hint: [look at the Snakefile](https://github.com/ngs-docs/2026-ggg-201b-rnaseq/blob/main/Snakefile)

### Run the analysis

Once the workflow succeeds, in RStudio, open the file `rnaseq-workflow.md` in the folder `201b-rnaseq` and select 'Knit to HTML' on the top menu bar.

That will take a few minutes.

et voila!

What's the output of _this_ process?

### Thoughts and comments on the overall RNAseq workflow

Here we are taking the results of _command line execution_ and feeding it into an RMarkdown document.

**RMarkdown** is a form of [literate programming](https://en.wikipedia.org/wiki/Literate_programming) that intersperses code with commentary and the outputs of the code.

The nice thing is that, if written properly, RMarkdown can be interpreted (and even potentially modified by) non-experts.

### How do you use the results of this?

Main pieces of advice:
- choose a p-adj/q-value threshold and ignore avg difference in expression; DESeq2 takes expression levels into account.
- then, use all genes with q-values below your threshold.

Q: what's the primary advantage of choosing a low q-value cutoff? What's the primary advantage of choosing a high q-value cutoff?

### Multiple testing and p- vs q-values

(If we have time)

Thought experiment (b/c titus forgot to bring coins).

Suppose I ask one of you to flip a coin five times in a row. What's the likelihood you get 5 heads?

Suppose I ask two of you to do the same. Now three. Now everyone. What is the likelihood that _someone_ in the class gets 5 heads?

::::spoiler
(Somewhere north of 50%)
::::

To deal with this, you apply multiple testing corrections - the simplest is [the Bonferroni correction](https://en.wikipedia.org/wiki/Bonferroni_correction), where you just divide by the number of trials, but that is quite stringent and reduces your sensitivity significantly. There are other techniques that can be used in specific circumstances; [here](https://bioconductor.org/packages//release/bioc/vignettes/DESeq2/inst/doc/DESeq2.html#independent-filtering-and-multiple-testing) is a description of what DESeq2 does.

### Plots output by this RNAseq workflow

* MDS plot
* MA plot
* individual genes

## Remainders

I've shown you many different _technical_ things:
* ondemand
* RStudio Server for remote computing
* Jupyter Lab for data exploration
* git to retrieve (and lightly modify) code
* conda for installing software

Bioinformatically, we've discussed:
* mapping
* assembly
* quantification and diff exp with RNAseq

All of these topics could use much more detail than I gave them, but hopefully you at least saw some interesteing stuff :)

the general bioinformatics vibe I want to communicate is:
- tracking what you've done is more important than getting it right the first 5 times, since you can always re-run things.
- any given technical detail can be figured out.
- build intuition about the process through parameter exploration and data investigation.
- think about positive/negative controls - do you have any? what about general expectations?
- think about cross validation more than you think about making the process "perfect"
- your bioinofrmatics will generally work out ok, if you pay attention to the process and take it seriously.

## What next?

Go about the normal business of grad school.

Find project(s). Do your QE. etc. etc.

Don't worry too much about skill building until you need to use the skill.

When you run into data analysis problems, come chat!

I hope to run [this class, BCB 298](https://docs.google.com/document/d/16UdjuCCAta-qdiDpFOS5qUaRTF_W-8ioiDLrA-QbFuU/edit?tab=t.0#heading=h.9dyxsn8t9et) indefinitely, and the hope is that this will be a place where you can come get pointers on what skills to learn and what processes to follow.

Live long and prosper!
