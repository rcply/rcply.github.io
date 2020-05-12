
# Assembling a transcriptome from short read data

## Downloading short read data

It's easiest to download fastq files from ENA, [the European Nucleotide Archive](https://www.ebi.ac.uk/ena). `https://www.ebi.ac.uk/ena`. You can also get them from the NCBI, but you need special tools and they come as an SRA file that needs additional processing to generate the fastq. You can search ENA with, for instance, a species name, bioproject ID, run names etc. Searching with a species name is likely to give several results, but we're looking for an experiment or run. Clicking through these should in the end lead to a table, one column of which is 'FASTQ files (ftp)'.

In my case, I find links to File 1 `ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR340/007/SRR3407257/SRR3407257_1.fastq.gz` and File 2 `ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR340/007/SRR3407257/SRR3407257_2.fastq.gz`. These can be downloaded with wget commands:

```bash
$ wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR340/007/SRR3407257/SRR3407257_1.fastq.gz
$ wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR340/007/SRR3407257/SRR3407257_2.fastq.gz
```

Trinity can be a bit funny about read names for paired end data. It likes to have the first read id ending in '/1' and the second in '/2'. You can check whether this is the case with commands like this:

```bash
$ gunzip -c SRR3407257_1.fastq.gz | head
```

And if need be you can fix with commands like these (you need to change the filenames):

```bash
$ gunzip -c SRR3407257_1.fastq.gz | perl -pe 's/^@(\S+)/\@$1\/1/' > left.fastq
$ gunzip -c SRR3407257_2.fastq.gz | perl -pe 's/^@(\S+)/\@$1\/2/' > right.fastq
```

## Cleaning transcriptome data

I'm not sure how much of a problem adaptor sequences cause. In principle it would obviously be better to remove them, but it's an extra step to mess up and it's not clear how much they really cause a problem for real downstream analysis. I don't know of any published studies, but [people get upset about it](http://www.opiniomics.org/we-need-to-stop-making-this-simple-fcking-mistake/). To identify adaptors you need to run fastqc:

```bash
$ fastqc <fastq file 1> <fastq file 2> <fastq file n>
```

and read the html file produced. You can then remove adaptors using trimmomatic parameters in Trinity (consult the Trinity manual).

## Assembling with Trinity

A typical trinity command looks like:

```bash
$ Trinity --seqType fq --max_memory 200G --left left.fastq --right right.fastq --CPU 10
```
where `--CPU` specifies the number of processors to use on your system and `--max_memory` the RAM you are allowing Trinity to use (if it needs it). 200G is a lot - 32G might be more realistic for a 'normal' server. Consult the manual for details of how to assemble from multiple fastq files and so on.

Depending on how much data there is / how fast your computer is, Trinity is likely to take hours or days to run. When it is finished, the assembled transcriptome is found in `trinity_out_dir/Trinity.fasta`

## How to search a transcriptome assembly for genes of interest

The transcriptome in `Trinity.fasta` is a bunch of nucleotide sequences. It's better to do database searches with protein sequences. There are programs that will conceptually translate the transcriptome during the search process. In the blast package from the NCBI, that means `tblastn`. To use BLAST, you first need to make a database using the `makeblastdb` command. The Fasta package lets you search a fasta file directly, without this additional step. To search `Trinity.fasta` with a protein query, you can use `tfasty` (the current version is called `tfasty36`):

```bash
$ tfasty36 my_query_protein.fasta Trinity.fasta > my_results.txt
```
For more large-scale analysis of proteins, it may be useful to predict open reading frames in the Trinity assembly, using [transdecoder](https://transdecoder.github.io/).
