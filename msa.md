
# Multiple sequence alignment

Database searches provide pairwise sequence alignments, between a query sequence and the database hit. Aligning many sequences in an optimal way is done using specialised Multiple Sequence Alignment (MSA) programs. Making an MSA is a necessary first step for phylogenetic analysis (for instance, the inference of species relationships, or accurate orthology inference). MSAs can also be used to produce statistical models of protein families (profiles, or hidden Markov models). These can then be used to detect more distant relationships between sequences than those found by BLAST or FASTA.

## Preparing data for multiple sequence alignment

You need to have a set of sequences in Fasta format. It's important to understand that any set of sequences can be aligned. It's your job to make sure that it makes sense to align them in the first place. This means being sure that they have something in common, usually statistically significant sequence similarity. This will be the case if you have gathered your sequences from a database search, provided you pay attention to the E-values of the hits to the sequences you extract.

## What to include?

The hard thing about multiple sequence alignment is knowing what to include. Unlike database search algorithms, multiple sequence aligners align over the entire length of the proteins that they are given. This can create problems if proteins only share a small region of sequence similarity relative to their overall length, as is often the case with multi-domain proteins Sequences that only include fragments of the thing that you're interested in can make it harder to interpret resulting alignments - is it a gap or a sequence fragment? Fragments caused by sequencing errors and incomplete gene prediction are not unusual in genome databases.

Depending on the exact situation, fixing these issues may be easier *after* the sequences have been aligned. Simple inspection of the results may make it obvious what cuts and trimming need to be made. In less favourable cases, these issues may mess up the alignment process, and it may not be easy to see what has happened. If you know what needs to go ahead of time though, and it's easy to do, you may end up with better results by pre-processing the sequences you include.

## Database searching tip - global/local search

Often when constructing a multiple sequence alignment, you know the region that you want to be covered, such as a single domain or combination of domains in a larger protein, or you might be interested in several repeating domains (e.g. a run of zinc fingers or WD40 repeats). These cases can cause lots of database hits to be fragments of the query sequence. One thing to try in this kind of  situation is the `glsearch36` program from the Fasta package. `glsearch36` performs a search that is global (i.e. full-length) in the query sequence, but local (i.e. can be partial) in the database sequence. You need to make a new fasta file from your query with just the region of interest in, then it's a drop in replacement for `ssearch36`:

```bash
glsearch36 region_of_interest.fasta database.fasta > results.txt
```

Beware that it will force alignments to be global on the query, even if they're bad matches. This should have the effect though, that good matches have much better scores and so can be easily extracted from the results.

## Sequence identifiers

You want to be able to interpret your sequence alignment after you've made it, so it helps to have informative sequence identifiers. This will depend on your question, but typically you might want to encode the species or gene family name in some way. Some alignment programs complain about repeated identifiers and some don't. Obviously you don't want repeated identifiers, but it may not matter if the program handles them sensibly. A related problem is that older programs sometimes truncated the sequence identifiers making them identical. This shouldn't happen with up-to-date alignment programs. Depending on the output format though, the identifiers might still be truncated in the output. Be careful.

## Making an alignment

The MAFFT program is a good choice for multiple sequence alignment. It can be downloaded [here](https://mafft.cbrc.jp/alignment/software/). Other choices include [muscle](https://drive5.com/muscle/) and [clustal omega](http://www.clustal.org/omega/), but below I'll use mafft.

Making an alignment is simple. If you have some sequences in a fasta file `my.fasta`:

```bash
mafft my.fasta > my.afa
```

This will produce an alignment in aligned fasta format. This is basically the same as fasta, but with gaps indicated by `-` characters, so all sequences are the same length. It's very portable and a good choice for subsequent computational analysis, but it's very difficult to see what's going on, so we'll instead try clustal format:

```bash
mafft --clustalout my.fasta > my.aln
```

This `my.aln` file can be looked at in the terminal with `less`. Sequences are now stacked on top of each other (i.e. aligned) to make the conserved positions obvious. The alignment comes in blocks of fixed width, with successive blocks printed after each other until the end of the alignment. Gaps are again shown with `-` characters and conserved columns are marked with an `*`.

It can be really helpful to colour in alignment columns to better highlight conserved features. [Jalview](http://www.jalview.org/getdown/release/) is a very good option for this. Easiest is to copy the alignment (with ctrl-c) and paste it into Jalview via the 'File->Input Alignment->From Textbox' menu selection. Jalview understands clustal and aligned fasta formats as well as others. I like the 'clustal' and 'Taylor' colour schemes for proteins in Jalview.
