---
tags: ggg, ggg2026, ggg201b
---

# Week 2 - Mapping and snakemake - GGG 201B lab section: Genomics (Winter 2026)

1/16/26

Titus Brown, ctbrown@ucdavis.edu

## At the beginning

* start zoom / recording!
* clean up directory & conda envs
* mention homework: will be posted today, due in two weeks.

Useful links:
* [An introduction to snakemake](https://ngs-docs.github.io/2023-snakemake-book-draft/)
* [Week 1 lab notes](https://hackmd.io/Kz38FYShRVKJiW3UI7IWtA?view)

## Learning objectives, Week 2:

At the end of today,
* you'll have downloaded a github repository to your local account
* you'll have installed snakemake and seen your first live Snakefile
* you'll have run your first snakemake workflow
* you'll have modified the workflow to properly record inputs and outputs so that snakemake can "chain" your workflow commands

But first...

## Start up RStudio Server on farm

Go to:

https://ondemand.farm.hpc.ucdavis.edu/

and log in.

Now select RStudio, and fill in the following values:

Account: ctbrowngrp
Partition: low
Number of cores: 2
Amount of memory: 5
Conda environment: r-4.4.2
Number of hours: 3

leave everything else as-is, and click "Launch".

## Go to the "Terminal" tab in RStudio

You should be  at a prompt that looks something like this:
```
ctbrown@cpu-3-50:~/$ 
```

Now let's check a few things, because Titus messed up last time...

## Check your conda `mapping` environment

```
module load conda
conda activate mapping
minimap2
```

what do you get?

BAD: `Command 'minimap2' not found, but can be installed ...`

GOOD: `Usage: minimap2 [options] <target.fa>|<target.idx> [query.fa] [...]`

### Maybe: fix conda.

If you get an error message above, try this:

::::spoiler
```
rm ~/.conda/envs/mapping
ln -s ~ctbrown/scratch3/2026-conda/$USER/mapping ~/.conda/envs

conda activate mapping
minimap2
```
::::

If _that_ returns an error, try:
::::spoiler

```bash
mkdir -p ~ctbrown/scratch3/2026-conda/$USER
mkdir -p ~/.conda/envs/
conda create -p ~ctbrown/scratch3/2026-conda/$USER/mapping minimap2 samtools bcftools

ln -s ~ctbrown/scratch3/2026-conda/$USER/mapping ~/.conda/envs
conda activate mapping
minimap2
```
::::

and if that doesn't work... flag me down :)

### You should now be at a prompt that looks like this:

```
(mapping) ctbrown@cpu-3-50:~/$
```

## Now! Grab a github repository

Let's copy all of [these files](https://github.com/ngs-docs/2026-ggg-201b-variant-calling) into our home directory:

```bash
git clone https://github.com/ngs-docs/2026-ggg-201b-variant-calling.git \
    ~/201b-variants
cd ~/201b-variants
```

and then let's link in some data files:

```bash
ln -s ~ctbrown/data/ggg201b/SRR2584857_1.fastq.gz ./
ln -s ~ctbrown/data/ggg201b/ecoli-rel606.fa.gz ./
```

Note: `ln -s` aliases them, rather than copying them; you could copy them with `cp` instead, but that would make extra copies.

## Install snakemake

Now let's install snakemake:

```bash
conda create -p ~ctbrown/scratch3/2026-conda/$USER/snakemake \
    -y snakemake-minimal
ln -s ~ctbrown/scratch3/2026-conda/$USER/snakemake \
    ~/.conda/envs
```

and activate the snakemake environment:
```
conda activate snakemake
```

Note: outside of class, this would just be:
```
XXX conda create -n snakemake -y snakemake-minimal
```
(I put the XXX there so that it will fail when you try to run it - you should not run this for THIS class.)

## Let's set up some more stuff commands

Create a directory:
```
mkdir -p outputs
```

and then:
```
snakemake -j 1 -p --use-conda uncompress_genome
snakemake -j 1 -p --use-conda map_reads
snakemake -j 1 -p --use-conda sam_to_bam
snakemake -j 1 -p --use-conda sort_bam
snakemake -j 1 -p --use-conda index_bam
snakemake -j 1 -p --use-conda call_variants
```

If either minimap2 or bcftools fails, see if they're installed:
```
conda activate mapping
minimap2
bcftools
```
(after this you'll want to run 
`conda activate snakemake` again.)

## Let's walk through these steps slowly.

Things to discuss:
* what is our "input" data, and what does it look like?
* where do we get reference genomes from?
* where did this data come from?
* what does each step do? why are there so many steps?
* what does `--use-conda` do, do you think?

Here is [the Snakefile we are using](https://github.com/ngs-docs/2026-ggg-201b-variant-calling/blob/main/Snakefile). You can also open it up in RStudio.

## Why snakemake? An initial set of thoughts.

Why not just run the commands like we did in week 1?? Why run them via snakemake:

If the command fails, it tells you clearly! With colors and everything!

You can activate different conda environments automatically. (This is very useful.)

It serves as a kind of "lab notebook" for your computational commands.

...but there's more!

## Annotate each rule with inputs and outputs

CTB note to self: check tabs.

For each rule, let's define an `input:` and an `output:` block.

At the end of this, you should be able to run:
```
rm -f outputs/*
snakemake -j 1 --use-conda call_variants
```
and have it work.

CTB note to self: paste intermediate and final steps in here.

### Final result:

```python!
rule uncompress_genome:
    input: "ecoli-rel606.fa.gz"
    output: "outputs/ecoli-rel606.fa"
    shell: """
        gunzip -c ecoli-rel606.fa.gz > outputs/ecoli-rel606.fa
    """

rule map_reads:
    input: "outputs/ecoli-rel606.fa", "SRR2584857_1.fastq.gz"
    output: "outputs/SRR2584857_1.x.ecoli-rel606.sam"
    conda: "mapping"
    shell: """
        minimap2 -ax sr outputs/ecoli-rel606.fa SRR2584857_1.fastq.gz > \
            outputs/SRR2584857_1.x.ecoli-rel606.sam
    """

rule sam_to_bam:
    input: "outputs/SRR2584857_1.x.ecoli-rel606.sam"
    output: "outputs/SRR2584857_1.x.ecoli-rel606.bam"
    conda: "mapping"
    shell: """
        samtools view -b outputs/SRR2584857_1.x.ecoli-rel606.sam > \
            outputs/SRR2584857_1.x.ecoli-rel606.bam
     """

rule sort_bam:
    input: "outputs/SRR2584857_1.x.ecoli-rel606.bam"
    output: "outputs/SRR2584857_1.x.ecoli-rel606.sorted.bam"
    conda: "mapping"
    shell: """
        samtools sort outputs/SRR2584857_1.x.ecoli-rel606.bam > \
            outputs/SRR2584857_1.x.ecoli-rel606.sorted.bam
    """

rule index_bam:
    input: "outputs/SRR2584857_1.x.ecoli-rel606.sorted.bam"
    output: "outputs/SRR2584857_1.x.ecoli-rel606.sorted.bam.bai"
    conda: "mapping"
    shell: """
        samtools index outputs/SRR2584857_1.x.ecoli-rel606.sorted.bam
    """

rule call_variants:
    input: "outputs/ecoli-rel606.fa", "outputs/SRR2584857_1.x.ecoli-rel606.sorted.bam"
    output: "outputs/SRR2584857_1.x.ecoli-rel606.pileup", "outputs/SRR2584857_1.x.ecoli-rel606.bcf", "outputs/SRR2584857_1.x.ecoli-rel606.vcf"
    conda: "mapping"
    shell: """
        bcftools mpileup -Ou -f outputs/ecoli-rel606.fa \
            outputs/SRR2584857_1.x.ecoli-rel606.sorted.bam  > \
            outputs/SRR2584857_1.x.ecoli-rel606.pileup
        bcftools call -mv -Ob outputs/SRR2584857_1.x.ecoli-rel606.pileup \
            -o outputs/SRR2584857_1.x.ecoli-rel606.bcf
        bcftools view outputs/SRR2584857_1.x.ecoli-rel606.bcf -o \
            outputs/SRR2584857_1.x.ecoli-rel606.vcf
    """

```

## Why snakemake? Some more thoughts

In addition to the above list, snakemake:
* can automatically "chain" rules
* can avoid re-running things unnecessarily

Next week, we'll talk about _templating_ and working with more than one data file.

## Alternative ways to automate things.

Workflow engines like snakemake are often a good choice, but they're not the only way to do this.

1. You can create a shell script - a file containing the commands you want to run - and run that.
2. You can also just do what I do sometimes and write down your commands in a hackmd.

Neither of these approaches will track files and avoid unnecessary commands, but they are useful in other ways.

## Why am I teaching snakemake?

Because:
- you need to know it exists
- it is not really that much more complicated, I don't think? but maybe I'm wrong
- it separates *what* you're running from *why* you're running. Shell commands are boring, results are interesting!

### Fini!

At the end of class, you can just close your browser and walk away, OR you can cancel your reservation in ondemand.

