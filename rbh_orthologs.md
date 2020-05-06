
## Reciprocal best hits, from the command line

If you think about the relationships between sequences in a phylogenetic tree, an orthologous pair of sequences should be closer to each other than they are to anything else: each other's best match in their respective genomes (ignoring complications from extra gene duplications). So we need to do 2 database searches and check that this relationships holds:

- Protein X (from genome B) vs (all proteins from genome A) gives best hit Protein Y.

- Protein Y (from genome A) vs (all proteins from genome B) gives best hit Protein X

This closer to each other than anything else is generally more
important that E-value thresholds *per se*.

We need to set up two sets of searches - the proteins from genome a vs the proteins from genome b, and *vice versa*. We're only interested in the best hit, because we're ignoring the possibility of paralogs (this is only a quick and dirty analysis).

I do this using `ssearch` from the FASTA package, but you could use BLAST. The important thing is that the 1st column of output contains the identifier of the searched sequence and the 2nd column, the database hit. In the commands below the `-m 8` option specifies this sort of tabular output. `-b 1` specifies just the best hit. `-E 0.1` is an E-value threshold, just to be on the safe side and guard against really spurious hits that happen to be also be 'reciprocal'.

```bash
ssearch36 -m 8 -b 1 -E 0.1 proteins_a proteins_b > proteins_a_vs_proteins_b_ssearch.txt
ssearch36 -m 8 -b 1 -E 0.1 proteins_a proteins_b > proteins_b_vs_proteins_a_ssearch.txt
```
From the 1st file (a vs b) extract just the identifiers, and sort them in alphabetical order:

```bash
awk '{ print $1,$2 }' proteins_a_vs_proteins_b_ssearch.out | sort -u > a_vs_b.txt
```

For the 2nd file (b vs a), we need to invert the identifiers so that we can compare the two files later. This is done by inverting the field specifiers in the awk command `$2` and `$1`:

```bash
awk '{ print $2,$1 }' proteins_b_vs_proteins_a_ssearch.out | sort -u > b_vs_a.txt
```

Each line consists of a pair of sequences. We just need to find the lines that are in common between the two files. We can do this using the unix `comm` command `comm` needs the files it's comparing to be sorted lexically, but we did this in the step above.

```bash
comm -12 a_vs_b.out b_vs_a.out > rbh_ortholog_pairs.txt
```

The -12 switch suppresses lines that are only found in file 1 or 2. `rbh_ortholog_pairs.txt` should now contain a list of reciprocal best hits, which are often used as an approximate set of orthologs.
