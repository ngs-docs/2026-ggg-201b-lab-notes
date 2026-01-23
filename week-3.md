---
tags: ggg, ggg2026, ggg201b
---

# Week 3 - More snakemake, more mapping - GGG 201B lab section: Genomics (Winter 2026)

1/23/26

Titus Brown, ctbrown@ucdavis.edu

Links:
- [Week 2](https://hackmd.io/h2CuD55HTpinqRLKvDhp0A?view)
- [snakemake book draft](https://ngs-docs.github.io/2023-snakemake-book-draft/)

## At the beginning

* clean up directory & conda envs
* start zoom / recording
* ask for final notebook from 201a... any copies out there??
* mention homework issues: not sure what's going on.

## Gentlepeople, start your RStudio

As usual, go to:

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

## Pick up from where we left off

In Terminal,

load conda and activate `snakemake` environment:
```
module load conda
conda activate snakemake
```

Change into the `201b-variants` directory:
```
cd ~/201b-variants
```

Clear your terminal (optional :):
```
clear
```

## some git stuff

As you may recall, at the end of last week we spent some time changing our Snakefile to make use of `input:` and `output:` to chain rules.

As a class, our Snakefiles may be in various states:
* unchanged!
* changed and not working!
* changed and working but not the same as mine! (which is where we'll start today)

Here's what I would like all your Snakefiles to look like: [link](https://github.com/ngs-docs/2026-ggg-201b-variant-calling/blob/main/Snakefile)

So the first thing we'll do is (a) 'stash away' any changes you've made to your copy, and (b) update your copy to match mine, by 'pulling' from my github repository, which I've updated.

Run:

```shell
# take away any changes you've made but not committed
git stash

# update from the original repository
git pull origin main
```

## snakemake & templating

### Recap: input and output annotations

Last Friday, we looked at annotating each rule with `input` and `output`. This gives snakemake the information it needs to:

* build missing intermediate files
* avoid re-running steps that are not needed

Let's look into that a bit more.

#### Reminder: the snakemake command line:

```
snakemake -j 2 --use-conda call_variants
```

You can restart from scratch by doing:
```
snakemake -j 2 --delete-all-output call_variants
snakemake -j 2 --use-conda call_variants
```

### The strange case of index_bam

Consider:
```python!
rule index_bam:
    input: "outputs/SRR2584857_1.x.ecoli-rel606.sorted.bam"
    output: "outputs/SRR2584857_1.x.ecoli-rel606.sorted.bam.bai"
    conda: "mapping"
    shell: """
        samtools index outputs/SRR2584857_1.x.ecoli-rel606.sorted.bam
    """
```

Question: what command outputs the .bai file?

And: what rule _uses_ the `.bai` file??

::::spoiler Answer:
It is _required_ for call_variants - the `samtools pileup` command uses the bai index! But it is not explicitly specified on the command line!

So it should be specified as an _input_ to call_variants even though it's not mentioned explicitly. If it's not specified, the rule will not be run!
::::

### The odd case of call_variants

Input and output annotations apply to the entire rule. The input files must exist _before_ the rule is run, and the output files should include all files _produced_ by the rule. That ends up making things a bit odd in the case of the rule call_variants, which runs multiple programs and produces multiple files.

I talked about this at the end of last week a bit - consider:

```python!
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

Here the file `"outputs/SRR2584857_1.x.ecoli-rel606.bcf"` is both an `output` of `samtools pileup` and an _input_ to `bcftools` view. But from snakemake's perspective, it's an output of the entire rule.

Why does this matter? Well let me show you some nonintuitive behavior :sweat_smile: 

### Making good use of input and output

#### First:

We can replace explicit mentions of filenames with templates. For example, we can take this rule:

```python!
rule uncompress_genome:
    input: "ecoli-rel606.fa.gz"
    output: "outputs/ecoli-rel606.fa"
    shell: """
        gunzip -c ecoli-rel606.fa.gz > outputs/ecoli-rel606.fa
    """
```

and update it to read:
```
rule uncompress_genome:
    input: "ecoli-rel606.fa.gz"
    output: "outputs/ecoli-rel606.fa"
    shell: """
        gunzip -c {input} > {output}
    """
```

and snakemake will happily replace `{input}` with whatever is in the `input:` block.

#### Second: multiple inputs/outputs

What do we do when we have multiple inputs or outputs?
```

rule map_reads:
    input: "outputs/ecoli-rel606.fa", "SRR2584857_1.fastq.gz"
    output: "outputs/SRR2584857_1.x.ecoli-rel606.sam"
    conda: "mapping"
    shell: """
        minimap2 -ax sr outputs/ecoli-rel606.fa SRR2584857_1.fastq.gz > \
            outputs/SRR2584857_1.x.ecoli-rel606.sam
    """
```

We can name them:
```

rule map_reads:
    input: genome="outputs/ecoli-rel606.fa", reads= "SRR2584857_1.fastq.gz"
    output: "outputs/SRR2584857_1.x.ecoli-rel606.sam"
    conda: "mapping"
    shell: """
        minimap2 -ax sr {input.genome} {input.reads} > \
            {output}
    """
```

Note that it's nice (but not necessary) to split the inputs across multiple lines:
```python!
rule map_reads:
    input:
        genome="outputs/ecoli-rel606.fa",
        reads= "SRR2584857_1.fastq.gz",
```

#### Third and trickiest:

```python!
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

What do we do here?

::::spoiler Answer

```python
rule call_variants:
    input:
        ref="outputs/ecoli-rel606.fa",
        bam="outputs/SRR2584857_1.x.ecoli-rel606.sorted.bam",
        bai="outputs/SRR2584857_1.x.ecoli-rel606.sorted.bam.bai",
    output:
        pileup="outputs/SRR2584857_1.x.ecoli-rel606.pileup",
        bcf="outputs/SRR2584857_1.x.ecoli-rel606.bcf",
        vcf="outputs/SRR2584857_1.x.ecoli-rel606.vcf",
    conda: "mapping"
    shell: """
        bcftools mpileup -Ou -f {input.ref} {input.bam} > {output.pileup}
        bcftools call -mv -Ob {output.pileup} -o {output.bcf}
        bcftools view {output.bcf} > {output.vcf}
    """
```

The main note here is that `input` and `output` are using the snakemake meaning (that is, they are inputs / outputs for the entire rule) not for the individual commands  so the `output.pileup` file is an output of `bcftools mpileup` but an _input_ to `bcftools call`
::::

### Going wild with wildcards

All of this seems mostly just annoying - why are we doing all this work??

Because, with one additional set of changes, we can use this snakemake workflow to analyze _many_ samples, not just one - by using _wildcards_.

tl;dr anywhere you see `ecoli-rel606` replace it with `{genome}`; anywhere you see `SRR2584857_1` replace it with `{sample}`. Here both 'genome' and 'sample' are wildcards that snakemake will infer automatically within each rule, based on requested files.

BUT: you now have a problem. snakemake doesn't "learn" the value of genome and sample; you need to give it a hint.

One way you can do this is by being explicit about the file name you want created on the command line. Instead of:

```
snakemake -j 2 call_variants
```
which will now complain about missing wildcard values, you can run:

```
snakemake -j 2 outputs/SRR2584857_1.x.ecoli-rel606.vcf
```

This should work, but is kind of annoying to write - can we make it nicer?

### Setting up a default output by requesting a default input

At the top, put:
```
SAMPLES = ["SRR2584857_1"]
GENOME = ["ecoli-rel606"]

rule make_vcf:
    input:
        expand("outputs/{sample}.x.{genome}.vcf",
               sample=SAMPLES, genome=GENOME),
```

This says, for _every_ named sample and _every_ named genome, make a VCF!

## A ~complete Snakefile

See [the homework 1 snakefile](https://github.com/ngs-docs/2026-ggg-201b-hw1/blob/main/Snakefile), which integrates all of the approaches above!

Update your snakefile by doing:
```
git stash
git pull origin main
```

OR mess yourself up by running:
```
cp ~ctbrown/201b-variants/Snakefile .
```

## Adding a new genome

Let's add a new E. coli genome - MG1655.

Here's the NCBI page for it: [link](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000005845.2/)

and here's the place you can get the files: [link](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/005/845/GCF_000005845.2_ASM584v2/)

I've downloaded them for you and placed them in my data directory, so you can get them by running:

```
cp ~ctbrown/data/ggg201b/ecoli-mg1655.fa.gz .
```

Let's update the rel606, too ;).
```
cp ~ctbrown/data/ggg201b/ecoli-rel606.fa.gz .

```

## Add `mg1655` to the GENOME list

change the GENOME line at the top of the Snakefile to:
```
GENOME = ["ecoli-rel606", "ecoli-mg1655"]

```


## Figuring out what variants mean: VEP

VEP stands for "Variant Effect Predictor". [link](https://www.ensembl.org/vep)

Install it like so:
```
conda create -n vep -y ensembl-vep
```

VEP requires a genome annotation, which we'll talk about how to generate next week or the week after :). For now, just copy it in like so:
```
cp ~ctbrown/data/ggg201b/ecoli-mg1655.sorted.gff.gz .
cp ~ctbrown/data/ggg201b/ecoli-rel606.sorted.gff.gz .
```

And we'll run it like so - add this to the Snakefile:
```python
rule tabix:
    input:
        gff="{filename}.gff.gz",
    output:
        tabix_idx='{filename}.gff.gz.tbi',
    conda: "mapping"
    shell: """
        tabix {input}
    """

rule predict_effects:
    input:
        fasta="{genome}.fa.gz",
        gff="{genome}.sorted.gff.gz",
        vcf="outputs/{reads}.x.{genome}.vcf",
        tabix_idx='{genome}.sorted.gff.gz.tbi',
    output:
        txt="outputs/{reads}.x.{genome}.vep.txt",
        html="outputs/{reads}.x.{genome}.vep.txt_summary.html",
        warn="outputs/{reads}.x.{genome}.vep.txt_warnings.txt",
    conda: "vep"
    shell: """
       vep --fasta {input.fasta} --gff {input.gff} -i {input.vcf} -o {output.txt}
    """
```

## If time: how can mapping go awry?

## homework and office hours

Note: homework is due next Thursday evening!

I have office hours at these times:

Monday, Jan 26th, 12-2pm

Thursday Jan 29th, 1-3pm

and I will post zoom links in Canvas for those times.
