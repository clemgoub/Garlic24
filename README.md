# GARLIC24

This repository tracks modifications made to the original [GARLIC](https://github.com/caballero/Garlic) repository in order to use it with the 2024 TE Hub benchmark on TE annotation. The fork `GARLIC24` was created by Clément Goubert.

The original work at the time of the fork is available under the branch `original`, which correspond also to the commit [`97cbb7e`](https://github.com/caballero/Garlic/commit/97cbb7e78eadd57748b97160b6f1ae0274237b6b) of the original repository.

## Features/Modifications:

- GARLIC now works with [DFAM's EMBL library](https://dfam.org/releases/Dfam_3.8/families/) instead of Repbase

## Documentation

### Input files for GARLIC24

| GARLIC script | file/argument | description | 
| ------------- | ---- | ----------- |
| **createModel.pl** | | | 
| | `-m model` | name given to the model; best if same as `genome`; also create dir: `./data/model` | 
| | `-f genome.fa` | genome in Fasta format | 
| | `-r genome.fa.align.gz` | RepeatMasker output (.align; `-a`); does not work with .out | 
| | `-t genome.fa.trf.bed` | TRF output for the genome; should match UCSC format |
| | `-g genome.table` | Gene annotations in old ensGene.txt format; from .gtf, use my R script `GTF2TXT.R` | 
| **createFakeSequence.pl** | | |
| | `-m model` | name given to the model; best if same as `genome`; expect to be found in `./data/model` |
| | `-f genome.fa` | |
| | `-s <size>` | size to generate; e.g. `1Mb`; throw a Warning about `Mb` but still works |
| | `-o` | output fake genome prefix; will add `.fasta` to it |
| | `--repbase_file` | EMBL formatted TE database; my modification of GARLIC works with DFAM 3.8 |

### Garlic Execution

#### `createModel.pl`

Let's define a working directory called `workdir/`. From now on we assume we are in `workdir/`
1. In `workdir/`, create a directory called `data` and add a subdirectory `model`
```sh
# in workdir/
mkdir data
mkdir data/model
```
2. Then, symlinc (or place) your input files in `data/`
```sh
# in workdir/data/model
ln -s <path/to/genome.fa> <genome.fa>
ln -s <path/to/genome.fa.trf.bed> <genome.fa.trf.bed>
ln -s <path/to/genome.fa.align.gz> <genome.fa.align.gz>
ln -s <path/to/genome.table> <genome.table>
```
3. Get back to the workdir and launch GARLIC
```sh
perl <path/to/Garlic24/bin/>createModel.pl -m <model> -f <genome>.fa -r <genome>.fa.align.gz -t <genome>.fa.trf.bed -g <genome>.table -v
```

#### `createFakeSequence.pl`

To execute in `workdir/`

```sh
perl <path/to/Garlic24/bin/>createFakeSequence.pl -m <model> -s <size_int>Mb -o <fake_genome_prefix> --repbase_file <DFAM_release>.embl -d ./data
```
> `-d ./data` is extremely important: it says where the files to create the fake sequence are. The default expect everything to be running within the `<path/to/Garlic24/bin/>` dir, which is not what we want.

-----

## Original documentation 

GARLIC - An artificial non-functional realistic DNA sequence generator.

Copyright (C) 2011-2015 Juan Caballero [Institute for Systems Biology]

DESCRIPTION

GARLIC is an artificial non-functional realistic DNA sequence generator.

Why do we need it? Because with current sequencing technology we can sequence 
any organism and start mining the genome. Many genomic analysis use strong 
statistical tests, but until now, a good negative control has never developed.

For example, we can use several programs (Genscan, Augustus, GlimmerHMM, ...) 
to predict coding genes, all these programs has been properly trained to 
recognize how a gene looks like using well characterized genes. So you expect a
low false negative rate, but none implements a negative control therefore you
will find a lot of false positive genes. This is currently true in many model 
organisms (including human) where the number of predicted coding genes are more 
than the number of coding genes with evidence. Of course you can use intergenic 
regions as a negative control, but these regions are limited in number and size,
also you cannot be 100% sure that the regions don't contain genes.

So, we developed this new tool to recreate realistic sequences based on the 
properties of the background genome. We define the background genome as the
remained sequences of a genome after removal of genes (coding and non-coding),
pseudogenes, interspersed repeats and low complexity sequences (600 Mb in hg19).

We modeled: 
(1) composition of the background genome
(2) interspersed repeats
(3) low complexity sequences

The current algorithm creates a base sequence, then the sequence is bombarded
with artificially evolved elements (interspersed repeats, low complexity) as
expected in the reference genome. The final output is a Fasta file with the 
new sequence generated.

REQUERIMENTS

- Perl
- Genome models. You can download from: http://www.repeatmasker.org/garlic/
  or create your own, see below.
- RepBase consensus sequences. You need to download the EMBL file from RepBase
  [http://www.girinst.org/repbase] (registration required) and put the file in
  data/repbase [suggested]. 

USAGE

1. Create a new sequence

  perl createFakeSequence.pl -m hg19 -s 1Mb -o fake.fa

For more options, please read the documentation using:

  perl createFakeSequence.pl --help

2. Train a new model

You can obtain the models from our website, we are currently suporting some
organisms with complete annotation in the UCSC Genome Database:
[http://hgdownload.cse.ucsc.edu/downloads.html]

You can create your own model fetching the data from the UCSC site:

  perl createModel.pl -m hg19

Also you can use your own sequences and annotations to create a model:

  perl createModel.pl -m myOrg -f myOrg.fa -r RM.out -t TRF.out -g Genes.table

Please read the documentation using:

  perl createModel.pl --help

CITATION

Realistic artificial DNA sequences as negative controls for computational genomics.
Caballero J, Smit AF, Hood L, Glusman G.
Nucl. Acids Res. 2014
doi: 10.1093/nar/gku356 

LICENSE

All the code is under the GPLv3 licence, see LICENSE file for details.


CHANGES

1.5 : 
  - "unitialized value" warnings caused by UCSC model lookups fixed.
  - TRF data should be in UCSC BED format which uses 0-based/half-open coordinates.
    Fixed a bug in the code where 1-based was assumed, causing a zero-valued start
    coordinate to go negative.
  - createModel.pl: Added support for relative paths in input parameters.



