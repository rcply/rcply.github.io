
# Searching a sequence database from the command line

How do you search a set of sequences that you have put together - for instance, a new transcriptome, or a collection of proteins from recently sequenced genomes that are not yet available in one of the big repositories? What if you've just read an exciting genome paper and want to find homologs of particular genes that you're interested in? This section is an introduction to how to do this from the unix command line - that is, from within a terminal application. I assume some basic familiarity with Unix, its commands and filesystem (how to move around between directories, move files, write files, redirection etc.)

## Sources of genomes

By genome, I really mean sets of predicted proteins. Interesting sources of data for marine invertebrates might include [OIST Marine Genomics Unit](https://marinegenomics.oist.jp/), [Ensembl Metazoa](http://metazoa.ensembl.org/index.html), [the NCBI genomes portal](https://www.ncbi.nlm.nih.gov/genome/browse#!/overview/) (filter this by 'animals', then 'other animals' unless you like vertebrates).

So, download a fasta file like [this one](https://marinegenomics.oist.jp/pau_v2/download/pau_genome_v2.0_prot.fa.gz) of the proteins from *Phoronis australiensis*. It's probably OK to download directly from the browser (right click, save as...), or you can do it from the command line with `wget`:

```bash
$ wget https://marinegenomics.oist.jp/pau_v2/download/pau_genome_v2.0_prot.fa.gz
```
or `curl`:

```bash
$ curl -O https://marinegenomics.oist.jp/pau_v2/download/pau_genome_v2.0_prot.fa.gz
```
if it's compressed, unzip it:

```bash
$ gunzip pau_genome_v2.0_prot.fa.gz
```
## Searching fasta files with FASTA

```
>Hs|HOXA1  gi|5031761|ref|NP_005513.1| gi|5031761|ref|NP_005513.1| homeobox protein Hox-A1 isoform a [Homo sapiens]
MDNARMNSFLEYPILSSGDSGTCSARAYPSDHRITTFQSCAVSANSCGGDDRFLVGRGVQ
IGSPHHHHHHHHHHPQPATYQTSGNLGVSYSHSSCGPSYGSQNFSAPYSPYALNQEADVS
GGYPQCAPAVYSGNLSSPMVQHHHHHQGYAGGAVGSPQYIHHSYGQEHQSLALATYNNSL
SPLHASHQEACRSPASETSSPAQTFDWMKVKRNPPKTGKVGEYGYLGQPNAVRTNFTTKQ
LTELEKEFHFNKYLTRARRVEIAASLQLNETQVKIWFQNRRMKQKKREKEGLLPISPATP
PGNDEKAEESSEKSSSSPCVPSPGSSTSDTLTTSH
```
The fasta format of sequence files (one line starting with a greater than sign and gene identifier, followed by lines of sequence) takes its name from the 'FASTA' package of sequence database search programs. These programs can be downloaded [here](https://faculty.virginia.edu/wrpearson/fasta/). As it says in the manual, the FASTA programs have several advantages over BLAST. They do not require any special preparation of the database to be searched, and they work better when searching a large database on a computer with not much memory. You can search an ordinary fasta file of the kind you downloaded above.

If you copy (i.e. ctrl-c or cmd-c) a sequence, you can write it to a file using the cat command, followed by ctrl-d:

```
$ cat > myseq.fasta
ctrl-v (to paste)
>Hs|HOXA1  gi|5031761|ref|NP_005513.1| gi|5031761|ref|NP_005513.1| homeobox protein Hox-A1 isoform a [Homo sapiens]
MDNARMNSFLEYPILSSGDSGTCSARAYPSDHRITTFQSCAVSANSCGGDDRFLVGRGVQ
IGSPHHHHHHHHHHPQPATYQTSGNLGVSYSHSSCGPSYGSQNFSAPYSPYALNQEADVS
GGYPQCAPAVYSGNLSSPMVQHHHHHQGYAGGAVGSPQYIHHSYGQEHQSLALATYNNSL
SPLHASHQEACRSPASETSSPAQTFDWMKVKRNPPKTGKVGEYGYLGQPNAVRTNFTTKQ
LTELEKEFHFNKYLTRARRVEIAASLQLNETQVKIWFQNRRMKQKKREKEGLLPISPATP
PGNDEKAEESSEKSSSSPCVPSPGSSTSDTLTTSH
ctrl-d (after pressing return to get a new line)
```

Less conveniently (if you are doing this a lot), you can open a text editor (e.g. vim, emacs or nano) and paste it in, saving it as a file.

Then we can search it against the fasta library file we downloaded. For a protein library, the best FASTA package program to use is `ssearch36`, for a nucleotide library, such as that made by `Trinity`, `tfasty36` works well.

Searching proteins:
```bash
$ ssearch36 myseq.fasta pau_genome_v2.0_prot.fa > myseq_results.txt
```
or, searching an assembled (but not translated) transcriptome:

```bash
$ tfasty36 myseq.fasta Trinity.fasta > myseq_results.txt
```
This should complete in a few seconds, and you can look at the results using a file viewer such as `less`. The results will look broadly similar in style to database search results on a web server, although there will be no links to external databases. After some preamble, there is a list of database 'hits', including scores and statistical significance, with the remainder of the file showing alignments and more summary scores of the more significant database hits.

## Retrieving sequences from a FASTA library

Here's the top of my hit list from the `ssearch` results. `blast` results look similar.

```
g5412.t1                                           ( 279)  398 77.6 8.6e-15
g5414.t1                                           ( 298)  326 65.1   5e-11
g6426.t1                                           ( 288)  317 63.6 1.4e-10
g5416.t1                                           ( 252)  304 61.4 5.8e-10
```

The left hand column has a list of sequence identifiers. You can search for these 'by hand' in the orginal Fasta file, but this can be a bit slow.

You can make a list of ids:

```
awk '{ print $1}' > id_list.txt
g5412.t1                                           ( 279)  398 77.6 8.6e-15
g5414.t1                                           ( 298)  326 65.1   5e-11
g6426.t1                                           ( 288)  317 63.6 1.4e-10
g5416.t1                                           ( 252)  304 61.4 5.8e-10
ctrl-d (after pressing return to get a new line)
```

This should give you a file `id_list.txt` with a single column of sequence identifiers. Again, you could use a text editor to do this.

To retrieve a list of ids from a Fasta database, you need to generate an index. You could do this with `samtools` or `esl-sfetch`. `samtools` is quite likely to be installed on bioinformatics servers (I tested with version 1.10). `esl-sfetch` is part of the easel tools in the hmmer package, which is fairly easy to install ([instructions](https://rcply.github.io/software_install.html)).

```bash
esl-sfetch --index pau_genome_v2.0_prot.fa
```
or:

```bash
samtools faidx pau_genome_v2.0_prot.fa
```
Then, depending on which tool you used to generate the index (they're not compatible with each other) you can query with a specific sequence identifier or a list of identifiers in a text file:

```bash
esl-sfetch pau_genome_v2.0_prot.fa g5414.t1
esl-sfetch -f pau_genome_v2.0_prot.fa my_list.txt
```
or (tested with samtools 1.10):

```bash
samtools faidx pau_genome_v2.0_prot.fa g5414.t1
samtools faidx pau_genome_v2.0_prot.fa --region-file my_list.txt
```

These commands will print the fasta sequences of the ids onto the screen. They can be redirected to a file using the `>` operator, to make a file of sequences that can be used for later analysis, such as multiple sequence alignment.
