
# Phylogeny for protein families

Most of the rancourous debate in phylogenetics revolves around species trees. When constructing a species tree, you generally have one 'sequence' per species, this being made up of the concatenated protein sequences of as many different genes as possible, while still having a reasonable alignment with the same genes from different species. This is a rather special use of phylogenetic analysis, aimed at answering the question 'what is the true phylogeny of these species?' Unless that happens to be your research question, I wouldn't get involved. If it is your research question start [here](https://www.nature.com/articles/s41576-020-0233-0).

In this section, I am concerned instead with the relationships between genes of a given family, with the possibility of many family members per species. How do we determine the species 'a' orthologs of genes from species 'b'? Recall that orthologs are genes that are related by speciation events, and paralogs genes that are related by a duplication event within a genome. In complex multi-gene families there are likely to be genes that are related by many such events, and a phylogenetic tree helps disentangle them.

### Making trees from multiple sequence alignments

People sometimes ask whether they should be using a Bayesian or Maximum Likelihood phylogeny program. I don't think it matters - the statistical framework should not be your major concern, but rather the model of evolution that you wish to use. In some cases this forces you to use a particular program, but for most typical cases of protein level gene phylogenies it's better to just choose a program which is easy to use and offers the models you want. Something like [`RAxML`](https://cme.h-its.org/exelixis/web/software/raxml/index.html) or [`iqtree`](http://www.iqtree.org/).

So, you need a multiple sequence alignment, and it should be an alignment of protein sequences, as these have richer evolutionary models available, and will yield more informative results. The number one tip for phylogeny reconstruction is to make sure your alignment is as good as it can be. You should look at it and remove sequences or regions of sequence that look obviously wrong, for instance, by not even remotely conforming to the consensus of all the other sequences. Regions where the alignment itself looks dubious should be removed - often this might correspond to the ends of the alignment, or to very 'gappy' regions showing little residue level conservation. Such edits are best done in an alignment editor like [Jalview](http://www.jalview.org/)

I think it's best to save the alignment in 'multiple fasta' format, as there is less scope to mess things up. (Things like clustal format can truncate sequence ids, which in turn can lead to duplicate ids, which can cause problems, either with the running itself or the interpretation of results).

```bash
iqtree -s my_sequences.mfa -nt AUTO -mset LG -pre my_tree
```

The big option for `iqtree` and phylogeny programs in general is what model of amino acid substitution to use. If you don't specify a model with `-m`, `iqtree` will test models to determine which best fits your data and use that. This can be quite a time consuming process if you have a big alignment. It generally picks some variant of the [LG](https://pubmed.ncbi.nlm.nih.gov/18367465/) model anyway, and these models are widely accepted to be good. A compromise option is to use `-mset LG` which just searches among the set of LG models for the best option, and skips other models (most of which it was never going to choose anyway).

To explain a few other things in the `iqtree` command: `-nt AUTO` tells the program to choose the optimum number of CPU threads to use for running, to speed things up. You can also specify this manually, but note that just setting a really high number probably won't work as well as you expect. `-pre` specifies the prefix for the files that `iqtree` writes. If you don't include this, it will use the alignment name, followed by `.treefile`, `.log`, `.mldist` and so on. Not a big deal.

A much faster option than `iqtree` is `FastTree`. This tends to produce less good results for more difficult cases, but is *very* much faster (like minutes vs many hours), and depending on your quesiton may be good enough.

```bash
FastTree -lg my_sequences.mfa > my_sequences.tree
```
Here `-lg` specifies the LG model of protein evolution (as we used for `iqtree`), probably a better choice than the default.

### Tree file formats

The standard tree format ('Newick') consists of sets of nested brackets representing tree nodes. For example ((a,b),c) says that 'a' and 'b' are children of a node to the exclusion of 'c'. This basic structure is then decorated with other information such as branch lengths, node names, bootstrap values and so on. Often this is on a single line. It's very difficult to 'read' this raw format by eye, so you need a tree viewer.

### Viewing trees

I use [FigTree](https://github.com/rambaut/figtree/releases). It's a good interactive viewer and handles large trees. The easiest way of loading a tree, I find, is to open a new file (Menu: File->New). This should give you blank FigTree window. You can then paste (i.e. ctrl-v) your tree directly into that window, and it should be displayed, graphically, as a tree.

On Mac's a good tip for this sort of thing (to avoid scrolling through terminals trying to select the required text for copying) is to use the command `pbcopy`. This loads the copy/paste buffer with the contents of `stdin`, which if you use a command like this:

```bash
cat my_tree.treefile | pbcopy
```
will be your tree file. You can then paste it using ctrl-v.

### The bootstrap and friends

How can you be confident about the relationships in your tree? One formal test is the bootstrap. This basically resamples (with replacement) columns from your alignment to make a new alignment of the same size, and then calculates a phylogenetic tree on the new alignment. Nodes that are present in the initial tree and also supported by this newly resampled alignment get a +1 on their boostrap score. This resampling is then repeated typically 100 or 1000 times, and the fraction of repeated nodes reported for each node on the original tree (so the boostrap never changes the original tree). This procedure is OK for small alignments where it is quick to calculate, but can be very slow, as you basically have to rebuild the tree 100 or more times.

A major complaint about the boostrap is that it can produce a false sense of certainty. If your evolutionary model is no good (and there is a lot we don't know about the way proteins evolve), branches can be well supported, but still wrong. *My* major complaint about it is that it doesn't really tell you anything that you can't (with a bit of experience perhaps) see for yourself in the structure of the tree. Well supported nodes tend to be well supported by the species evidence (i.e. genes from similar species behave in similar ways). Well supported nodes tend to be separated by relatively long braches. When branches are tightly spaced, they don't have time to accumulate many amino acid substitutions, so it's less likely that there will be good support for associated tree nodes. Journals like bootstrap values though. For `iqtree` you invoke them with `-b 100` where the number is the number of replicates. `iqtree` also has an 'ultrafast' bootstrap that can be invoked with `-B 1000`.
