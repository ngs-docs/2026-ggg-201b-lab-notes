---
tags: ggg, ggg2026, ggg201b
---
# Lab Homework #3 GGG 201b - Mar 9, 2026

Due by 10pm, Tuesday March 17th, 2026

For this homework, you're going to add some samples to [the RNAseq analysis pipeline we'll use in week 10](https://hackmd.io/7KKH1C3XThCa2vSV4oxzcg?view#A-simple-RNAseq-workflow-on-yeast).
This RNAseq workflow conducts a differential expression analysis of three wild-type (wt) yeast data sets against three snf2 knockouts.

For HW #3, you'll add add two new data sets into the workflow, and adjust the p-value cutoff to be 0.05 instead of 0.1. 

## 1. Install software, claim the homework, clone, and run.

First follow the instructions [in the lab notes for week 10](https://hackmd.io/7KKH1C3XThCa2vSV4oxzcg?view#A-simple-RNAseq-workflow-on-yeast), through the end of "install the software". If you've already done this in class, you don't need to repeat it - you just need the `2026-rnaseq` conda environment to exist. After installation, shut down the RStudio.

Next, claim hw #3 [here](https://classroom.github.com/a/moHGdHyK).

Then, start a new RStudio via ondemand in the `2026-rnaseq` conda environment.

At the Terminal, clone your homework repository into your farm account using the ssh URL; then, load the conda module, and activate the `2026-rnaseq` conda environment. Run the workflow at the terminal in the homework directory like so:

```
snakemake -c 4
```

## 2. Make edits etc.

In your running RStudio, do the following four steps:

(a) edit the Snakefile and add the links to the two new data sets to the SAMPLES dictionary at the top:

ERR458496 - URL https://osf.io/tzagu/download

ERR458503 - URL https://osf.io/px7sf/download

(You can see more about the data sets 
[here](https://www.ebi.ac.uk/ena/browser/view/ERR458496?show=reads) and [here](https://www.ebi.ac.uk/ena/browser/view/ERR458503?show=reads)).

(b) edit `rnaseq_samples.csv` to load the new sample `quant.sf` files that will be produced by the Snakefile; note that ERR458496 is `wt` and ERR458503 is `snf2`.

(c\) Edit line 230 of `rnaseq-workflow.Rmd` to set the p-value cutoff to be 0.1.

Next, run the snakemake workflow with
```
snakemake -j 4
```
and once it completes, load `rnaseq-workflow.Rmd` into RStudio and knit it to HTML.

Verify that your HTML workflow report shows the new data sets in the PCA and the MDS plots, and that the individual gene report has four points for each condition.

## 4. Commit your changes to four files and push to github

You'll have four changed files to preserve and hand in. They are:

* `Snakefile`
* `rnaseq_samples.csv`
* `rnaseq-workflow.Rmd`
* `rnaseq-workflow.html`

To commit and push them, do
```
git commit -am "hw3"
git push
```

Check to make sure that the updated `rnaseq_samples.csv`, Rmd file, and `Snakefile` are all up on your github.

And then... you're done! Congrats!