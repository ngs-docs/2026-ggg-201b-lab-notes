---
tags: ggg, ggg2026, ggg201b
---

# Week 5 - De novo assembly! - GGG 201B lab section: Genomics (Winter 2026)

2/6/26

Titus Brown, ctbrown@ucdavis.edu

## At the beginning

* start zoom / recording!
* clean up directory & conda envs
* mention homework and next week guest lecture

## In-class assembly exercise

Working individually or in groups, please reconstruct the original text for the handouts I’m giving you (and posting on Canvas). Please don't use a search engine, but otherwise use the computer as you wish. I do suggest you avoid any attempts at programming...

Specific questions:

* what techniques or tactics are you using?
* suppose you had to do this with lots more data; how would you do it?
* suppose you were given an unlimited supply of undergraduates; what would you tell them to do and how would you split the work up?


what advantages do you have in this exercise over DNA?

what are the pluses and minuses of the different datatypes?

Does the order or organization of reads matter?

if you get to use a reference, how do you know it’s the right reference?
* (what strategies might you use to validate your assembly?)
* (talk about standard computational approach)

what strategy would you use if I told you you could have as many undergrads as you wanted to do this?

**Note:** If you are remote, please feel free to work together or separately - I'll mute you in the classroom for the moment, and check in with you periodically to see if you have any questions!

I'll give you 30 minutes :). Your objective is not _necessarily_ to actually reconstruct the text, but every so often people actually do...

## Discussion!!

More info goes here.

Prompt:
> * what techniques or tactics are you using?
> * suppose you had to do this with lots more data; how would you do it?
> * suppose you were given an unlimited supply of undergraduates; what would you tell them to do and how would you split the work up?
> 
> 
> what advantages do you have in this exercise over DNA?
> 
> what are the pluses and minuses of the different datatypes?
> 
> Does the order or organization of reads matter?
> 
> if you get to use a reference, how do you know it’s the right reference?
> * (what strategies might you use to validate your assembly?)
> * (talk about standard computational approach)
> 
> what strategy would you use if I told you you could have as many undergrads as you wanted to do this?

## Learning objectives, hands-on section of week 5:

At the end of today,
* you'll have installed the megahit assembler, the quast assembly metrics analysis package, and the prokka bacterial annotation software
* you'll have done an assembly and annotation of an E. coli genome
* you'll have learned how to measure the computational requirements of a task

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

## Stats for the assembly

Let's take a look at the assembly statistics, as calculated by quast:

```
cat SRR2584857_quast/report.txt
```

## Next class

We will look at the annotation process a bit, and also think about coverage and such.
