
# Quantifying reads for RNA-seq using STAR

## Quantifying reads in general

There are a couple of issues to consider. Firstly are you interested in genes or transcripts? Secondly do you have a genome? The recipe here is for the genes / genomes pair, using [STAR](https://github.com/alexdobin/STAR). This provides an easy entry into subsequent analysis of differential expression using e.g. the [DESeq2](https://bioconductor.org/packages/release/bioc/html/DESeq2.html) R package, but there are alternative pipelines, such as [kallisto](https://pachterlab.github.io/kallisto/about) and [sleuth](https://pachterlab.github.io/sleuth/about), that may better suit your purposes. There is a big literature on accurately inferring differential gene expression - it is a good idea to be familiar with why you are making the choices you're making.

## Using STAR

The [STAR manual](https://github.com/alexdobin/STAR/blob/master/doc/STARmanual.pdf) is good.

I find it helps keep things in order to make three directories for STAR analyses. One to hold the fastq reads to be mapped, one to hold the genome, and one to do the analysis in. This isn't essential, but there tend to be a lot of files in complex analyses.

For this example, I'm going to assume that we're working in a base directory with three subdirectories called `fastq/`, `genome/` and `star_mapping/`

## Making a genome database for STAR

Copy the files `clytia_hm2` (or your genome fasta file), `filtered_for_orf.gtf` (or your gene annotation file) and `mk_index.com` to a new genome directory.

Run the command in `mk_index.com`.

```
STAR --genomeDir . --genomeFastaFiles clytia_hm2 --sjdbGTFfile filtered_for_orf.gtf --genomeChrBinNbits 16 --genomeSAindexNbases 14 --runMode genomeGenerate

```

This will take a while - it combines the annotation in the gtf file with the sequence in the genome file to make a database that STAR can search. The `--genomeChrBinNbits 16` and `--genomeSAindexNbases 14` relate to issues searching small and fragmentary genome and are discussed in the STAR manual. When it is done there should be many new files in the working directory.

Generally you only need to do this once, although I have seen STAR complain that the database version has changed (i.e. if you use a new version of STAR, you may need to recreate the database).

## Making a directory of fastq files

In your base directory, make a directory called `fastq`. It doesn't have to be called that, but in my examples, that is its name. Copy fastq files to be mapped to this directory. They can be compressed. For this example I have copied in a set of paired end reads `Clytia-aboral-1_1.fq.gz` and `Clytia-aboral-1_2.fq.gz`

## Mapping with STAR

In the base directory make a directory called `star_mapping`. (Again the name isn't important). `cd` into that directory. We're now ready to do the mapping and quantification. The final command is a bit complicated so we're going to build it up gradually here. The basic form of the command is:

```
STAR --genomeDir <genome_directory> --readFilesIn <left_reads_file> <right_reads_file> --quantMode GeneCounts
```

To get this to actually work, we need to tell STAR where to find the genome directory and read files (as we've put them in a different directory) - so, like this:

```
STAR --genomeDir ../genome/ --readFilesIn ../fastq/Clytia-aboral-1_1.fq.gz ../fastq/Clytia-aboral-1_2.fq.gz
```

Now, these read files are compressed, so we need to tell STAR how to uncompress them...

```
STAR --genomeDir ../genome/ --readFilesIn ../fastq/Clytia-aboral-1_1.fq.gz ../fastq/Clytia-aboral-1_2.fq.gz --readFilesCommand gunzip -c
```

and we want it to count genes...

```
STAR --genomeDir ../genome/ --readFilesIn ../fastq/Clytia-aboral-1_1.fq.gz ../fastq/Clytia-aboral-1_2.fq.gz --readFilesCommand gunzip -c --quantMode GeneCounts
```

Read mapping is easily paralllelized, so we'll specify extra CPUs to get it going faster, and provide an output file prefix so that we can easily identify our output in a useful way:

```
STAR --genomeDir ../genome/ --readFilesIn ../fastq/Clytia-aboral-1_1.fq.gz ../fastq/Clytia-aboral-1_2.fq.gz --readFilesCommand gunzip -c --quantMode GeneCounts --runThreadN 32 --outFileNamePrefix Ch_AB_1_
```

This is a bit incidental at this point, but he main output of STAR is a very big `.sam` file. We specify that it be compressed to a `.bam` file:

```
STAR --genomeDir ../genome/ --readFilesIn ../fastq/Clytia-aboral-1_1.fq.gz ../fastq/Clytia-aboral-1_2.fq.gz --readFilesCommand gunzip -c --quantMode GeneCounts --runThreadN 32 --outFileNamePrefix Ch_AB_1_ --outSAMtype BAM Unsorted
```

## STAR output files

Check the final log file (`*__Log.final.out`) - in our case `Ch_AB_1_Log.final.out`. The key thing to check is the % of uniquely mapped reads. You would like this to be above 60% ideally. The exact figure might depend on the experimental system, how much contamination there is etc. Very low numbers (e.g. close to 0%) suggest that something has gone wrong e.g. you have accidentally mixed up read files / not given correct paired reads.

The gene counts are in the `*_ReadsPerGene.out.tab` file, `Ch_AB_1_ReadsPerGene.out.tab`. This contains 4 columns: 1) the gene identifier 2) the total number of reads for the gene 3) the number of left reads 4) the number of right reads. (The left or right reads may be close to 0 if the sequenced library was made using a stranded protocol).
