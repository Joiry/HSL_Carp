# Peak Calling and Multiomic Comparisons

* Peak calling
* How to call peaks with MACS
* Comparing interval data Bedtools
* Visulization

* 
* 
* 

***

## Peak Calling

Peak calling is a technique used for various types of DNA sequencing where specific areas of the genome have been isolated.

Some examples:

* [ChIP-Seq](https://en.wikipedia.org/wiki/ChIP_sequencing) - regions isolated by use of antibodies targeting proteins in DNA binding complexes

* [FAIRE-Seq](https://en.wikipedia.org/wiki/FAIRE-Seq) - use of formaldehyde to cross-link open areas of chromatin

* [ATAC-Seq](https://en.wikipedia.org/wiki/ATAC-seq) - uses a transposase to insert sequencing adapters in open chromatin


As we can see in the [Workflow Overview](https://joiry.github.io/HSL_Carp/workflow), the preceding steps for these types of NNN-Seq data also follow the common early steps of QC and trimming, and then alignment to the reference genome.  Note, for these sorts of targeted DNA sequencing, we must always align to the full genome and not a subset like only the transcriptome.

I've already aligned a set of FAIRE-Seq data for us to use.  These are big, but not too large, so we'd best work in our /pine/scr/ space for these

~~~
$ mkdir faireseq
$ cd faireseq/
$ cp /proj/seq/data/carpentry/faire_data/*bam .
~~~


Note, I've added in some annotation on what each of these are:
https://www.ncbi.nlm.nih.gov/Traces/study/?acc=PRJNA383482&o=acc_s%3Aa

~~~
SRR5457501_1.bam      # 3LW
SRR5457502_1.bam      # 3LW
SRR5457503_1.bam      # 3LW
SRR5457504_1.bam      # 24hAPF
SRR5457505_1.bam      # 24hAPF
SRR5457506_1.bam      # 24hAPF
SRR5457507_1.bam      # 44hAPF
SRR5457508_1.bam      # 44hAPF
SRR5457509_1.bam      # 44hAPF
~~~


[MACS project page](https://github.com/macs3-project/MACS)



***  

## Using MACS2



~~~
$ module load macs
$ module list
~~~

~~~
Currently Loaded Modules:
  1) python/2.7.12   2) macs/2.1.2
~~~


~~~
$ cp /proj/seq/data/carpentry/scripts/*macs2*sh .
$ ls -1 *.sh
~~~


There are four scripts we're going to use.  The first is a general purpose script that can call peaks on each one of our samples.

~~~
$ less s_macs2_single.sh
~~~

~~~
#!/bin/bash
#
#SBATCH -p general
#SBATCH -N 1
#SBATCH -n 1
#SBATCH --mem 2g
#SBATCH -t 1:00:00 # time (D-HH:MM)
#SBATCH -o macs2.%x.%j.out 

bam=$1
base=$(basename $bam .bam)

macs2 callpeak -n $base -g dm \
      -t $bam \
      --broad --outdir faire_${base}_out
~~~


The script can be used one at a time, but is also structed to be loop friendly:

*(note, you could turn the for loop into int's own script, which then calls the macs2 script.  Scripts calling other scripts is a common thing to do)*

~~~
$ for bamfile in $(ls -1 *bam); do sbatch -J $bamfile s_macs2_single.sh $bamfile; done
$ squeue -u tristand
~~~

The next three combine the three replicates of each condition to allow for more data to call the peaks.  This can be useful at times, but isn't always necessary.  This is another decision that basically has to be made depending on the system and how the data shakes out.

Let's go ahead and run the three other scripts, and then take a look at how they differ:

~~~
$ sbatch -J 3LW s_macs2_3LW.sh
$ sbatch -J 24hAPF s_macs2_24hAPF.sh
$ sbatch -J 44hAPF s_macs2_44hAPF.sh
~~~

Now that everything is running, let's look at how these three scripts differ from the generic one.

~~~
$ less s_macs2_3LW.sh 
~~~

While the sbatch headers remain the same, I've made changes to the callpeaks run.  It would be tempting to make one generic script for having multiple bams in one callpeaks run, but we run into issues with trying to pass a list of files into a script.  It's doable, but making a script for an arbitrary list of bam files might not be worth your time, while creating one with a specified number of bams is possible, but limiting (eg $bam1 $bam2 $bam3).

~~~
macs2 callpeak -n faire_3LW -g dm \
      -t SRR5457501_1.bam SRR5457502_1.bam SRR5457503_1.bam  \
      --broad --outdir faire_3LW_out
~~~

Once these runs complete, we can look into the resulting files:

~~~
$ cd faire_SRR5457501_1_out
$ ls -1
~~~

~~~
SRR5457501_1_model.r
SRR5457501_1_peaks.broadPeak
SRR5457501_1_peaks.gappedPeak
SRR5457501_1_peaks.xls
~~~

We can use the command `wc` to see how many peaks we have, in fact using wildcards, if we just up a directory we can get a quick report on all peak calling attempts due to how we've named the resulting files.

~~~
$ cd ..
$ wc -l faire*out/*broad*
~~~


***

## Bedtools

[Bedtools](https://bedtools.readthedocs.io/en/latest/) is a powerful suite of programs that can perform various comparisons with interval data, typicall Bed format files, but also can work with BAM, GFF, and VCF (Variant Call Format) files.

~~~
$ module load bedtools
$ module list
~~~

~~~
Currently Loaded Modules:
  1) python/2.7.12   2) macs/2.1.2   3) samtools/1.16   4) bedtools/2.30
~~~

Since bedtools depends on samtools, and the modules system has be set to automatically load samtools as well.

[Tools of Bedtools](https://bedtools.readthedocs.io/en/latest/content/bedtools-suite.html) - we can see that Bedtools 40 different subcommands, we're only going to scratch the surface, looking at the `intersect` tool.

Of import note in using `interset`, and any of the other Bedtools, is the "A" (`-a`) file is compared to the "B" (`-b').  The A file is fully loaded into memory, so all other factors being equal, it's generally a good idea to use the smaller of the files as the A file.

So, we're going to need to files to compare, one will be a peaks file we generated.

The other will the gene annotation file we used in generating counts for the DESeq analysis, `/proj/seq/data/dm6_UCSC/Annotation/Genes/genes.gtf`, which brings us a step closer to comparing two sets of "-omics" data.  Let's go ahead an make a local copy of this GFF file and give it a slightly more descriptive name.

~~~
$ cp /proj/seq/data/dm6_UCSC/Annotation/Genes/genes.gtf dm6_genes.gtf
$ less -S dm6_genes.gtf
~~~

Now, we don't necessarily want this entire file to compare intervals with, so we can use `grep` to narrow it down.  Recall that when we did gene counts, `featureCounts` only used the "exon" entries.  We don't have to limit ourselves to exons tho, we may for example be interested in peaks that only overlap the start codons for example.  In ChIP-Seq analyses, often you'll want a custom GFF file that only has the first kilobase or more preceding the TSS if you are looking for proximal promoter regions.

~~~
$ grep "exon" dm6_genes.gtf > dm6_genes_exons.gtf
$ grep "start_codon" dm6_genes.gtf > dm6_genes_starts.gtf
~~~

~~~
$ sbatch -o bed_inter.exons_vs_24hAPF.%j.out --wrap="bedtools intersect -a dm6_genes_exons.gtf -b faire_24hAPF_out/faire_24hAPF_peaks.broadPeak -wa -wb  > bed_inter.exons_vs_24hAPF.txt"
~~~

Once it finishes, we can look at the resulting file

~~~
$ less -S bed_inter.exons_vs_24hAPF.txt
~~~

You can see this is no longer a properly formated GFF nor Bed file, and you'd have to do some custom script writing, probably in a language more powerful than just the shell commands, like Python or Perl, to do further analysis.  And here we've only looked at the one set of peaks.  In practive, you'll spend a lot of time comparing all your peak sets to various intervals, depending on the exact type of "-seq" data and the overall goals of the experiment.


## Visualizing data with IGV


[Integrative Genomics Viewer IGV](https://software.broadinstitute.org/software/igv/) - as it's name suggests, can be used to graphically view the data in Bam, Bed, and other file formats of genomic data.



**[Interactive Demo of IGV with files generated from lessons]**

