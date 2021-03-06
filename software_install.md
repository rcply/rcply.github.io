
## Installing software on a Mac or Unix-like system

This can be a difficult and frustrating business. These notes are just for the easy cases. Most cases are easy, especially for popular software (one of the reasons that software might be popular is that it's easy to install), but when things don't work, there are so many possible things that might have gone wrong that you'll need to get advice from an expert or Google the error message.

## Is the program you want compiled already?

If the software you want gives you the opportunity to download precompiled binaries, it is generally easier to do this. Binaries are specific to machine types, so you're likely to see versions for OSX, Linux and Windows. Download the correct version, unpack it and you should be OK. Remember that any program you want to run needs to be in your path, or you need to specify the path to that program directly when you invoke it. e.g. if you had a program called `run_me` in your `Downloads` folder, you'd have to type something like:

```bash
~/Downloads/run_me
```
If this sort of thing is completely unfamiliar it might be a good idea to investigate some basic Unix tutorials.

## Compiling programs

Some software is distributed as source code (a text file with the program written in it) and needs compiling before it can be used. Ordinary Unix systems generally are set up to allow compilation of programs - you can skip to the next section. If you're using a Mac, you may need to install a compiler first.

### Do you have a compiler installed already (for Mac)?

You need to open a 'Terminal'. You can find the Terminal program from the Finder, following:

```
Macintosh HD -> Applications -> Utilities -> Terminal
```
The icon is a little TV screen with a '>' in the top left corner. Open it.

Not all software needs to be compiled, but a lot does. Check if you have a C compiler installed by typing `gcc`.

```bash
$ gcc
```

If you get an error message like:

```
$ gcc
clang: error: no input files
```

you *do* have a compiler. If, not, you need to install the Apple developer tools. You could install 'Xcode' from the App store, but this is enormous and takes ages to download. It's possible the computer will prompt you what to do after typing gcc, but if not you can  install the command line tools:

```bash
xcode-select --install
```

### Compile the HMMER software

Download the latest [HMMER](http://hmmer.org/download.html) code from http://hmmer.org/download.html. HMMER is a set of programs for performing sensitive searches of protein sequence databases using Hidden Markov Models of protein sequence families.

You could right/ctrl-click on the link and 'Download linked file' or you could use curl:

```bash
curl -O http://eddylab.org/software/hmmer/hmmer-3.3.tar.gz
```

`.tar.gz` files are a typical way of distributing Unix software source code. The `tar` file is an archive (it stands for 'Tape ARchive', from the time when things were stored on tapes) - a single file with many files stored inside, and the `.gz` part is `gzip` compression. You can unpack the entire thing with:

```bash
tar -zvxf hmmer-3.3.tar.gz
```

`-v` means 'verbose' and tar will tell you all the files it's extracting. Probably everything is fine, unless you can see an explicit error message at the end.

In general it's then a question of following the installation instructions, most likely in a file called `INSTALL` or `README`, or on the web page from which you downloaded. In the case of hmmer:

```bash
cd hmmer-3.3
./configure --prefix /your/install/path
make
make check
make install
```
The `--prefix /your/install/path` is optional, if you don't want to use the place that hmmer wants to use. If you don't change it, you may have to type in your password at the install stage. This is OK.

I recommend also installing the 'easel' tools as there is some useful stuff there - in particular `esl-sfetch` is a good tool for pulling sequences out of fasta databases.

```bash
cd easel
make install
```
You can check whether it has worked - type `phmmer`:

```
% phmmer
Incorrect number of command line arguments.
Usage: phmmer [-options] <seqfile> <seqdb>

where basic options are:
  -h : show brief help on version and usage

To see more help on available options, do phmmer -h
```
If you see this, it *has* worked. If you get `phmmer: command not found`, open a new Terminal and try again (just the typing `phmmer`, not the whole install). If it still hasn't worked, get advice from your local source of advice.
