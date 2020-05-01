
# How to search a sequence database from the command line

How do you search a set of sequences that you have put together - for instance, a new transcriptome, or a collection of proteins from recently sequenced genomes that are not yet available in one of the big repositories? What if you've just read an exciting genome paper and want to find homologs of particular genes that you're interested in? This section is an introduction to how to do this from the unix command line - that is, from within a terminal application. I assume some basic familiarity with Unix and its filesystem (how to move around between directories, move files etc.)

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
The fasta format of sequence files (one line starting with a greater than sign and gene identifier, followed by lines of sequence) takes its name from the 'FASTA' package of sequence database search programs. These programs can be downloaded [here](https://faculty.virginia.edu/wrpearson/fasta/). As it says in the manual, the FASTA programs have several advantages over BLAST, not least the fact that they do not require any special preparation of the database to be searched. You can search an ordinary fasta file of the kind you downloaded above.
