---
tags: ggg, ggg2026, ggg201b
---

# Week 4 - Shotgun sequencing, mapping, and references - GGG 201B lab section: Genomics (Winter 2026)

1/30/26

Titus Brown, ctbrown@ucdavis.edu

Links:
- [Week 3](https://hackmd.io/5DRf9snFSuiugJLm5mCp_A?view)

## At the beginning

* start zoom / recording
* I would like to do an in-class exercise next week. It is not critical but is kind of fun. Please let me know (by e-mail) if you know you _cannot_ make it in person on Feb 6th.

Also: we're not going to do much (any) computer work today, it's all going to be discussion and "whiteboarding".

## Homework & VEP problems

This year I tried to introduce better/more useful interpretation of variants by using the Ensembl [Variant Effect Predictor](https://www.ensembl.org/vep).

It didn't work well - most of your variants were "intergenic" according to VEP, right?

That is because VEP relies on an annotation in a "gene feature format" file, and the [NCBI gene feature format files are incompatible with VEP](https://github.com/Ensembl/ensembl-vep/issues/1620).

As an aside, I figured this out _not_ because I dug into the REL606 results, but because of the MG1655 reference analysis we did at the end of class last week.

So I spent the morning borrowing and modifying code to [fix the GFF files](https://github.com/ctb/2026-sort-gff).

Let me show you the results.

If you re-copy over the GFF files, and rerun your homework analysis, you'll get different results.

Morals of the story:
* bioinformatics formats are annoying
* a simple control can help you figure out when you're doing something wrong

## How do you measure quality of mapping?

A simple set of mapping stats can be generated with the `samtools flagstat` command:
```
samtools flagstat outputs/SRR2584403_1.x.ecoli-rel606.bam.sorted
```

which generates:
```
2001307 + 0 in total (QC-passed reads + QC-failed reads)
2001176 + 0 primary
0 + 0 secondary
131 + 0 supplementary
0 + 0 duplicates
0 + 0 primary duplicates
1736355 + 0 mapped (86.76% : N/A)
1736224 + 0 primary mapped (86.76% : N/A)
0 + 0 paired in sequencing
0 + 0 read1
0 + 0 read2
0 + 0 properly paired (N/A : N/A)
0 + 0 with itself and mate mapped
0 + 0 singletons (N/A : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```
How should we interpret this?

Is this better or worse? (You won't have this file in your homework)
```
% samtools flagstat outputs/SRR2584403_1.x.ecoli-mg1655.bam.sorted
2002514 + 0 in total (QC-passed reads + QC-failed reads)
2001176 + 0 primary
0 + 0 secondary
1338 + 0 supplementary
0 + 0 duplicates
0 + 0 primary duplicates
1586760 + 0 mapped (79.24% : N/A)
1585422 + 0 primary mapped (79.22% : N/A)
0 + 0 paired in sequencing
0 + 0 read1
0 + 0 read2
0 + 0 properly paired (N/A : N/A)
0 + 0 with itself and mate mapped
0 + 0 singletons (N/A : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```

Questions to discuss:

* what could be going "wrong" here?
* how do you know if your reference genome is "good" for your reads??
* what do you do if it's not??

## Topics to cover

shotgun sequencing!
* "Shotgun" !?
* "Random"

Short reads:
* PCR based
* error profile
* paired end

Long reads:
* error profile

Coverage

Variant calling:
* what are we doing? pipeline revisit. viz revisit.
* role of coverage; where is it reported?
* low coverage variant calling
* joint variant calling

