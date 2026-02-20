---
tags: ggg, ggg2026, ggg201b
---

# Week 7 - De novo assembly, take2! - GGG 201B lab section: Genomics (Winter 2026)

2/20/26

Titus Brown, ctbrown@ucdavis.edu

## At the beginning

* start zoom / recording!
* clean up directory & conda envs
* next week: guest lecture on RNAseq

## Learning objectives, hands-on section of week 7:

At the end of today,
* you'll have installed the megahit assembler, the quast assembly metrics analysis package, and the prokka bacterial annotation software
* you'll have done an assembly and annotation of an E. coli genome
* you'll have learned how to measure the computational requirements for a task

But first...

## Start up RStudio Server on farm

Go to:

https://ondemand.farm.hpc.ucdavis.edu/

and log in.

Now select RStudio, and fill in the following values: (**note: 10 GB, 8 cores**)

Account: ctbrowngrp
Partition: low
**Number of cores: 8**
**Amount of memory: 10**
Conda environment: r-4.4.2
Number of hours: 3

leave everything else as-is, and click "Launch".

## Go to the "Terminal" tab in RStudio

You should be  at a prompt that looks something like this:
```
ctbrown@cpu-3-50:~/$ 
```

## Load conda

```
module load conda
```

## Create a few conda environments

```bash
conda create -p ~ctbrown/scratch3/2026-conda/$USER/megahit \
    -y megahit quast

conda create -p ~ctbrown/scratch3/2026-conda/$USER/prokka \
    -y prokka

conda create -p ~ctbrown/scratch3/2026-conda/$USER/sourmash \
    -y sourmash

ln -s ~ctbrown/scratch3/2026-conda/$USER/{megahit,prokka,sourmash} \
    ~/.conda/envs
```

### Cleaning up old disk space

If you run out of disk space with the above, it is because despite my best efforts, conda is putting the downloaded packages in your home directory under `~/.conda`. These can safely be removed - run
```
conda clean --all
```
to do so, and then re-run any failed commands.

You can also remove the variant calling directory which is reasonably large:
```
rm -fr ~/201b-variants
```

If you run into problems with a particular conda environment install that failed the first time you ran it, just remove the entire directory like so:
```
rm -fr ~ctbrown/scratch3/2026-conda/$USER/prokka
```
and then re-run conda.

### Measuring disk space usage

The command
```
du -sk ~/.??* ~/* | sort -n
```
will tell you what is using how much disk space in your home directory.

The command:
```
df -h ~/
```
will tell you how much free disk space you have to work with.

## Activate the snakemake conda environment

```bash!
conda activate snakemake
```

### You should now be at a prompt that looks like this:

```
(snakemake) ctbrown@cpu-3-50:~/$
```

## Now! Grab a github repository

We are going to use a new snakemake workflow
to do de novo assembly: [link](https://github.com/ngs-docs/2026-ggg-201b-assembly)

Let's copy all of [these files](https://github.com/ngs-docs/2026-ggg-201b-assembly) into our home directory:

```bash
git clone https://github.com/ngs-docs/2026-ggg-201b-assembly.git \
    ~/201b-assembly
cd ~/201b-assembly
```

## We need some files!

Now let's link in some data files:

```bash
ln -s ~ctbrown/data/ggg201b/SRR2584857_?.fastq.gz .
```

Note: '?' in the command, what ends up in the directory?

## Check that we have everything ready to go:

Try doing a "dry run" with snakemake:
```
snakemake -n
```

What does it report?

Dry runs are useful for debugging - snakemake will tell you what it's going to run, without actually running it.

## Run snakemake, look at the outputs

```
/usr/bin/time -v snakemake -j 8 --use-conda
```

It will take about 5 minutes and at the end you should see something like this:

```
       Command being timed: "snakemake -j 8 --use-conda"
        User time (seconds): 845.95
        System time (seconds): 27.35
        Percent of CPU this job got: 293%
        Elapsed (wall clock) time (h:mm:ss or m:ss): 4:57.08
        Average shared text size (kbytes): 0
        Average unshared data size (kbytes): 0
        Average stack size (kbytes): 0
        Average total size (kbytes): 0
        Maximum resident set size (kbytes): 913920
        Average resident set size (kbytes): 0
        Major (requiring I/O) page faults: 4
        Minor (reclaiming a frame) page faults: 2987910
        Voluntary context switches: 117661
        Involuntary context switches: 99176
        Swaps: 0
        File system inputs: 831576
        File system outputs: 2523368
        Socket messages sent: 0
        Socket messages received: 0
        Signals delivered: 0
        Page size (bytes): 4096
        Exit status: 0
```

So, this next bit is kind of a digression from assembly, but it's a pretty important computational point so I'm going to discuss it :).

Let's look at the following lines:


```
User time (seconds): 845.95
System time (seconds): 27.35
Percent of CPU this job got: 293%
Elapsed (wall clock) time (h:mm:ss or m:ss): 4:57.08
Maximum resident set size (kbytes): 913920
```

The fourth line is how long it took, in minutes/seconds, according to the actual clock on the wall. So, you waited ~5 minutes.

The fifth line is how much memory it required. This is essentailly a lower limit on how much memory you need to allocate via  ondemand to run this workflow - in this case, about 1 GB (1 million kilobytes).

The first three lines are trickier to explain, but it boils down to, in order:
* how much CPU time was used by the user, to do read comparisons etc;
* how much CPU time was used by the system, to do things like read/write from the disk;
* how _many_ CPUs were used - in this case, about 3, because ~300%.

So anyway this is how you figure out how "expensive" a particular task or set of tasks (in this case, the entire workflow) is. And this can guide how much memory and time you ask to reserve via OnDemand.

## Examining the assembly itself

The primary output of this job is a bunch of assembled DNA contigs.

Run:
```
head SRR2584857-assembly.fa
```

what do you see?

Yeah, not that useful :laughing: 


## Annotation

The next step after genome assembly is genome _annotation_. Last week Dr. Soto showed you how to _view_ and explore annotated genome features, but first we have to generate them!

Let's do so by running the [prokka](https://github.com/tseemann/prokka) tool.

```
snakemake --use-conda prokka_assembly -j 4
```


Honestly, reading the text output of this is fascinating in terms of understanding the process! (This is saved in the .log file below).

The important / interesting files output by prokka are all under the directory `SRR2584857_annot` and are:
* SRR2584857_annot.txt - summary of output
* SRR2584857_annot.err - summary of errors
* SRR2584857_annot.faa - **protein sequences**
* SRR2584857_annot.ffn - **the gene sequences**
* SRR2584857_annot.gff - **GFF file containing features!**
* SRR2584857_annot.tsv - spreadsheet of genes / annotations

Additional files:
* SRR2584857_annot.fna - the original assembly file, unchanged (but renamed :)
* SRR2584857_annot.fsa - annotated assembly file (contigs named)
* SRR2584857_annot.gbk - genbank format annotation
* SRR2584857_annot.log - log of the output
* SRR2584857_annot.sqn - not sure exactly
* SRR2584857_annot.tbl - some other format

### Annotation basics

Genome annotation is almost entirely based on:
* simple models of genes, e.g. open reading frames (ORFs);
* homology (inferred by sequence similarity) using either BLAST or HMMER;
* other data sets (especially: RNAseq)
* less reliable: eukaryotic gene finders

Open reading frames are really simple: you find ATGs followed by long stretches of non-stop-codons, and you call it a gene!

### How annotation differs in euks and bacteria/archaea/viruses

tl;dr Introns suck.

Bacterial and viral gene prediction mostly Just Works, except when biological weirdness gets in the way (e.g. non-canonical amino acids).

If you are annotating the genes in a eukaryotic genome, *especially* one with no close relatives that are well annotated, you will almost always want good comprehensive RNAseq.

Here one big problem is that RNAseq is always of a particular tissue/mixture of tissues/life stage, so genes that are not expressed then/there won't be represented. There is also the problem that transcription from the genome is pretty noisy, so there are a lot of places that are transcribed but that (don't seem to be) genes. Often, protein-coding is used as a filter.

But if you _just_ use sequence similarity, you almost always miss UTRs.

So it always ends up being a mixture of computational techniques.

Check out the following two papers for the actual process used to construct genes anotations from data -

* Figure 1 of [Tissue resolved, gene structure refined equine transcriptome](https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-016-3451-2)
* Figure 1 in [Identification of long non-coding RNA in the horse transcriptome](https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-017-3884-2)

## Stats for the assembly

Let's take a look at the assembly statistics, as calculated by quast.

Let's look at 
```
cat SRR2584857_quast/report.txt
```

what does it tell us?

In RStudio, open the file `icarus.html`. Let's explore the contig length distribution.

Topics to cover -
* N50 and L50
* why not just "mean"?

## Evaluating coverage

The coverage formula for average coverage from shotgun sequencing is pretty simple:

$$
C = B / G
$$
where R is the total number of bases sequenced and G is the estimated genome size.

The total number of bases sequenced can be broken down into:
$$
B = R * L
$$
where R is the number of reads and L is the average length of reads.

To get the number of reads in a FASTQ file, you can run:
```
gunzip -c SRR2584857_1.fastq.gz | wc -l
```
and then divide that number by 4.

To get your average read length, you can either run something like FastQC or just look at the FASTQ file like so:
```
gunzip -c SRR2584857_1.fastq.gz | head
```

G for this genome is about 4.5m. What's the coverage of these reads?

### Evaluate coverage with k-mers

Let's use k-mers to look at the reads. Wee're going to use [sourmash](https://sourmash.readthedocs.io/) for this.

First, use sourmash to turn the reads and assembly into k-mers:

```
snakemake --use-conda sketch_reads sketch_assembly -j 4 -p
```

Next, activate the sourmash environment install the abundhist plugin:
```
conda activate sourmash
pip install sourmash_plugin_abundhist
```

and then run it like so:
```
sourmash scripts abundhist SRR2584857-reads.sig.zip --figure abundhist.png
```

you can also try:
```
sourmash scripts abundhist SRR2584857-reads.sig.zip \
    -I SRR2584857-assembly.sig.zip \
    --figure abundhist-genome.png
```

What do these figures show?

Note that you can adjust the number of bins by adding `--bins`, e.g. `--bins 50`.

Do these plots agree or disagree with your coverage estimate from just straight up math??
