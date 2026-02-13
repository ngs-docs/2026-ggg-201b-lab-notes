---
tags: ggg, ggg2026, ggg201b
---

[toc] 

# Lab Homework #2 GGG 201b - Feb 12, 2026

Due: 9pm on Thursday, Feb 26th

Titus office hours in DataLab (Shields 360/362): 
* Monday, Feb 23rd, 12-2pm
* Wed, Feb 25th, 11am-1pm


Please note that you may freely work together with others, but you must hand it in separately.

---

## Goals

This homework starts from [Lab 5](https://hackmd.io/vU7QkySoSPmJD6OU8m6S2g?view) to do a bit more with assembly.

The goal of this homework is to examine and evaluate at _what coverage_ genome assembly collapses. To this end, you will calculate assemblies for several different subset sizes of reads using a Snakefile, and report the result to a central form.

Warning: the computation here will take about 20-30 minutes to run once you have the Snakefile all set up.

## 0. Make sure your farm account is setup properly

You'll need to run the conda installation commands from [Lab 5](https://hackmd.io/vU7QkySoSPmJD6OU8m6S2g?view) before doing this homework.

## 1. Connect to GitHub Classroom and clone the hw2 repository

As per [Lab homework #1](https://hackmd.io/csNHtwWQTT2M7ccbfjD9oQ?view), connect to [Lab homework #2](https://classroom.github.com/a/QaIiTRHI) and clone this assignment's repository to farm.

You'll also need to [link in the data files](https://hackmd.io/vU7QkySoSPmJD6OU8m6S2g?view#We-need-some-files).

## 2. Edit the Snakefile and save to GitHub

Update the Snakefile to calculate at least three different subset assemblies, choosing estimated coverages between 2x and 60x. The default target for the Snakefile should start from a directory containing only the two read files (`SRR2584857_1.fastq.gz` and `SRR2584857_2.fastq.gz`) and the Snakefile, and compute all three assemblies plus their quast statistics and prokka annotations.

To change the size of the read files that are being assembled, adjust the number "400000" at the very top to represent different size subsets.
```
rule all:
    input:
        "SRR2584857_quast.4000000",
        "SRR2584857_annot.4000000",
```
If you want to calculate multiple subsets at the same time, you can just add a pair of lines for each subset:
```
rule all:
    input:
        "SRR2584857_quast.4000000", # 4m lines
        "SRR2584857_annot.4000000",
        "SRR2584857_quast.3000000", # 3m lines
        "SRR2584857_annot.3000000",
        "SRR2584857_quast.2000000", # 2m lines
        "SRR2584857_annot.2000000",
```

Get the Snakefile working, verify that it works in an empty directory (using `--delete-all-output`), and then commit and push, as in homework 1. Please do not  add the read data files or any other files into the git repository or GitHub, thanks!!

If you work with others, please select different subset sizes between all of you. Thanks!! (It won't take any extra time ;)

Note, you'll need to add `conda:` blocks with the appropriate conda environment names to this Snakefile; use [this repo](https://hackmd.io/vU7QkySoSPmJD6OU8m6S2g?view#We-need-some-files) for inspiration!

## 3. Submit assembly results to this Google form

Submit at least three _different_ assembly entries to [this form](https://forms.gle/8uXko9831Q2moQv39), which will ask for the following information:
1. Your GitHub ID.
2. The filename of one assemby produced by your Snakefile.
3. The number of reads used in the assembly.
4. Your estimate of the coverage (use 4.5 Mb as the genome size).
5. The N50 of the assembly.
6. The total bp in contigs > 1kb for the assembly.
7. The total number of contigs > 1kb for the assembly.
8. The total number of protein coding genes (records in the annotation .faa file) for the assembly.

Note that the total number of lines in a single gzipped FASTQ file can be calculated like so:
```
gunzip -c FILENAME | wc -l
```

and the total number of records in a FASTA file (e.g. the .faa file output by prokka) can be calculated by doing
```
grep ^'>' FILENAME | wc -l
```
You can inspect the records in a FASTA file with
```
gunzip -c FILENAME | head
```
which will let you calculate the sequence length - note that all the sequences in the FASTQ file are the same length.

## 4. Relax in knowledge of a job well done.

Please do inspect the Snakefile on github via the Web interface to make sure it has all your changes!

**You must turn this in via GitHub, and I will primarily be looking at GitHub for grading.**

## Questions? Problems?

If you run into any trouble with submission, that's ok - just let me know.

You can reach me via e-mail at ctbrown@ucdavis.edu, or in UC Davis slack under @ctitusbrown.

## Appendix - snakemake details

If you're interested in understanding some of what's going on in the Snakefile, here is some reading!

### wildcards and expand

General background reading on what we're doing with wildcards in this Snakefile: [Wildcards are determined by the desired output](https://ngs-docs.github.io/2023-snakemake-book-draft/beginner+/wildcards.html#wildcards-are-determined-by-the-desired-output).

### "|| true"

The `|| true` in the `subset_reads` rule is explained in this draft chapter: [Never fail me - how to make shell commands always succeed](https://ngs-docs.github.io/2023-snakemake-book-draft/recipes/never-fail-me.html?highlight=exit%20codes#never-fail-me---how-to-make-shell-commands-always-succeed).

### `{subset}` - how it works and what it's used for

In the `subset_reads` rule, there's some tricky stuff with `{subset}` being used to specify the number of lines being extracted with `head`. See [Using wildcards to determine parameters to use in the shell block](https://ngs-docs.github.io/2023-snakemake-book-draft/beginner+/wildcards.html#using-wildcards-to-determine-parameters-to-use-in-the-shell-block) for an explanation of this.