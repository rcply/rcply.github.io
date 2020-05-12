
# How to search a sequence database from the command line

How do you search a set of sequences that you have put together - for instance, a new transcriptome, or a collection of proteins from recently sequenced genomes that are not yet available in one of the big repositories? What if you've just read an exciting genome paper and want to find homologs of particular genes that you're interested in? This section is an introduction to how to do this from the unix command line - that is, from within a terminal application. I assume some basic familiarity with Unix, its commands and filesystem (how to move around between directories, move files, write files, redirection etc.)

## Sources of genomes

By genome, I really mean sets of predicted proteins. Interesting sources of data for marine invertebrates might include [OIST Marine Genomics Unit](https://marinegenomics.oist.jp/), [Ensembl Metazoa](http://metazoa.ensembl.org/index.html), [the NCBI genomes portal](https://www.ncbi.nlm.nih.gov/genome/browse#!/overview/) (filter this by 'animals', then 'other animals' unless you like vertebrates).

So, download a fasta file like [this one](https://marinegenomics.oist.jp/pau_v2/download/pau_genome_v2.0_prot.fa.gz) of the proteins from *Phoronis australiensis*. It's probably OK to download directly from the browser (right click, save as...), or you can do it from the command line with `wget`:
```bash
wget https://marinegenomics.oist.jp/pau_v2/download/pau_genome_v2.0_prot.fa.gz
```
or `curl`:
```bash
curl -O https://marinegenomics.oist.jp/pau_v2/download/pau_genome_v2.0_prot.fa.gz
```
if it's compressed, unzip it:

```bash
gunzip pau_genome_v2.0_prot.fa.gz
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

If you copy (i.e. ctrl-c or cmd-c) a sequence, you can write it to a file using the cat command, followed by ctrl-D:

```
cat > myseq.fasta
ctrl-v (to paste)
>Hs|HOXA1  gi|5031761|ref|NP_005513.1| gi|5031761|ref|NP_005513.1| homeobox protein Hox-A1 isoform a [Homo sapiens]
MDNARMNSFLEYPILSSGDSGTCSARAYPSDHRITTFQSCAVSANSCGGDDRFLVGRGVQ
IGSPHHHHHHHHHHPQPATYQTSGNLGVSYSHSSCGPSYGSQNFSAPYSPYALNQEADVS
GGYPQCAPAVYSGNLSSPMVQHHHHHQGYAGGAVGSPQYIHHSYGQEHQSLALATYNNSL
SPLHASHQEACRSPASETSSPAQTFDWMKVKRNPPKTGKVGEYGYLGQPNAVRTNFTTKQ
LTELEKEFHFNKYLTRARRVEIAASLQLNETQVKIWFQNRRMKQKKREKEGLLPISPATP
PGNDEKAEESSEKSSSSPCVPSPGSSTSDTLTTSH
ctrl-D (after pressing return to get a new line)
```

Less conveniently (if you are doing this a lot), you can open a text editor (e.g. vim, emacs or nano) and paste it in, saving it as a file.

Then we can search it against the fasta library file we downloaded:

```bash
ssearch36 myseq.fasta pau_genome_v2.0_prot.fa > myseq_results.txt
```
This should complete in a few seconds, and you can look at the results using a file viewer such as `less`. The results will look broadly similar in style to database search results on a web server, although there will be no links to external databases. After some preamble, there is a list of database 'hits', including scores and statistical significance, with the remainder of the file showing alignments and more summary scores of the more significant database hits.
