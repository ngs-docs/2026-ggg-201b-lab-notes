---
tags: ggg, ggg2026, ggg201b
---

# Week 1 - Mapping - GGG 201B lab section: Genomics (Winter 2026)

1/9/26

Titus Brown, ctbrown@ucdavis.edu

## At the beginning

* start zoom / recording
* clean up my apps & disk
* what's up with the "section" thing?? tl;dr canvas
* apologize for 201a overlap, which is actually fairly minimal

## Introducing myself

Bioinformatics generalist.
Methods developer.
_Mostly_ focused on shotgun sequencing data interpretation, _mostly_ of microbiome data, _mostly_ through workflow & methods development.

## Learning objectives, Day 1:

At the end of today,
* you'll have been introduced to the scope and requirements for the lab section of the course;
* you will be able to allocate resources on the farm HPC and start up RStudio;
* you will have gotten a very limited and basic introduction to the UNIX command line;
* you will have installed some software and gotten some data;
* you will have seen the mapping workflow;

(The real goal is actually just to make sure you're set up to run stuff, but the rest is important too!)

## Syllabus

Let's go through the [syllabus for lab section of 201B](https://hackmd.io/YUWAdpnjSBGhMuej_MIjVg?view)

## Bioinformatics, genomics, data science, and computing

What are they?

Three sets of threes:
- the three circles of data science (=> bioinformatics): science domain, math/statistics, practical knowledge of how to do something.
- the three types of bioinformaticians: biomedical data scientist, methods developer, and workflow-enabled bioinformatician.
- the three practical skillsets you need: scripting, interpreting, organizing

What do you need to know?

How do you learn it?

How will you know you've learned it?

My perspective/suggestion: view this through the lens of **community of practice**. It's just like knitting or dancing: you've got to actively do it in order to learn it.

If you are doing computing, embed yourself in a community of practice - your lab, your grad group, the Genome Center, DataLab, or online.

Above all, note you can't just take a class NOW in preparation for doing data analysis in 6 or 12 or 18 months. This will introduce you to the concepts and some of the practice, but you will almost certainly need a strong refresher before you can get started. That's OK! I'll tell you more about that at the end.

## Starting up RStudio Server on farm

### First things first: Connecting to OnDemand

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

Note: `low` is a low-priority queue that lets you allocate jobs under 4 hours across the entire cluster, so it's good for class! You might want to use high for doing homeworks.

### Why OnDemand?

It's a really convenient way to connect to HPCs, and it lets you view files, edit files, run programs, and look at results all in a Web browser.

### Exploring OnDemand: the files tab

Let's start by looking at the files tab!

You probably don't have much here just yet, but this is a storage space  just for you (20 GB by default).

You can upload and download files here, too.

### Connecting to RStudio and using the shell

File system window
R console
Editor window
Shell/terminal window

### The shell or Terminal window

Shell commands are "verb noun".

A few shell commands:

```
pwd
mkdir ~/201b-mapping
cd ~/201b-mapping
ls
pwd
echo 'stuff for day 1 class' > README.txt
```

## Running your first mapping!

Make sure you are in the Terminal, or at the shell prompt. It should look something like:
`ctbrown@gpu-10-50:~$ ` but with your own username and with a different machine name than 'gpu-10-50'.

### Step 1: Installing software with conda

Activate the conda software module:
```
module load conda
```

Create a necessary directory:
```
mkdir -p ~/.conda/envs
```

Run a convoluted command so that your software installs are not in your home directory but rather use my lab's disk space:
```
conda create -p ~ctbrown/scratch3/2026-conda/$USER/mapping -y minimap2 samtools bcftools 
ln -s ~ctbrown/scratch3/2025-conda/$USER/mapping ~/.conda/envs
conda activate mapping
```

(Normally this would be much simpler:`conda create -n mapping -y minimap2 samtools bcftools`, followed by `conda activate mapping`.)

Q: How do you know what software to install?

### Step 2: Copy in some data

```
cd ~/201b-mapping

cp ~ctbrown/data/ggg201b/SRR2584857_1.fastq.gz .
```

Q: How big is the file? What's in it?

We can look at this data with:
```
gunzip -c SRR2584857_1.fastq.gz  | head
```

Q: where did this file come from?

## Step 3: Copy a reference genome into our directory

```
cp ~ctbrown/data/ggg201b/ecoli-rel606.fa.gz .
```

Q: How big is the file? What's in it?

Again, we can look at .gz files like so:
```
gunzip -c ecoli-rel606.fa.gz | head
```

Q: where do reference genomes come from??

## Step 4: Run the basic mapping of reads to reference; summarize mapping & call variants

(We will go through all of this again next Friday :)

Run these commands one at a time; `#` denotes a comment (won't do anything if entered).
```
# map reads; produce a SAM file
minimap2 -ax sr ecoli-rel606.fa.gz SRR2584857_1.fastq.gz  > SRR2584857_1.x.ecoli-rel606.sam

# convert SAM to a binary BAM file for efficient storage & manipulation
samtools view -b SRR2584857_1.x.ecoli-rel606.sam > SRR2584857_1.x.ecoli-rel606.bam

# sort the BAM file by position in genome
samtools sort SRR2584857_1.x.ecoli-rel606.bam > SRR2584857_1.x.ecoli-rel606.bam.sorted

# index the file
samtools index SRR2584857_1.x.ecoli-rel606.bam.sorted

# produce a summary of the "piled up" reads
bcftools mpileup -Ou -f ecoli-rel606.fa.gz SRR2584857_1.x.ecoli-rel606.bam.sorted > SRR2584857_1.x.ecoli-rel606.pileup

# "call" variants and produce a binary call file (BCF)
bcftools call -mv -Ob SRR2584857_1.x.ecoli-rel606.pileup -o SRR2584857_1.x.ecoli-rel606.bcf

# convert to a text file that we can look at
bcftools view SRR2584857_1.x.ecoli-rel606.bcf > SRR2584857_1.x.ecoli-rel606.vcf
```

## Step 5: Inspect the called variants

The net result of this workflow is this file; scan the contents like so:
```
grep -v ^# SRR2584857_1.x.ecoli-rel606.vcf
```

Q: why so few variants!?

To look at the lined up reads, run:
```
samtools tview SRR2584857_1.x.ecoli-rel606.bam.sorted ecoli-rel606.fa.gz -p ecoli:1
```
replacing `1` with the location of a region of interest.

Use `q` to exit; arrows to move around; `.` to convert between views.

### Fini!

You can just close your browser and walk away, OR you can cancel your reservation in ondemand.

## GGG 201(b) links

[Links to some UNIX tutorials](https://github.com/ngs-docs/2025-ggg-201b-lab/blob/main/unix-tutorials.md)