# pbptyper
In silico Penicillin Binding Protein (PBP) typer for _Streptococcus pneumoniae_ assemblies

## A Quick Note

The original [CDC StrepLab](https://www.cdc.gov/streplab/pneumococcus/mic.html) implementation is
available at [BenJamesMetcalf/Spn_Scripts_Reference](https://github.com/BenJamesMetcalf/Spn_Scripts_Reference) and
[BenJamesMetcalf/JanOw_Dependencies](https://github.com/BenJamesMetcalf/JanOw_Dependencies).

I'll be the first to admit, my Perl skills aren't what they used to be. I tried. But I also wanted to
simplify things a little. The original method uses FASTQs as inputs, then trims adapters, assembles with Velvet,
predicts genes, Blast genes, etc... I think these days its much easier to just start with assemblies.
I changed things so that, all that's needed is an assembly and `tblastn` to predict the PBP type.

## Introduction

`pbptyper` is a tool to identify the Penicillin Binding Protein (PBP) of _Streptococcus pneumoniae_ assemblies.
Using an input assembly (uncompressed or gzip-compressed), the PBP 1A, 2B, and 2X proteins are blasted against
the assembly from which a PBP type is predicted.

The PBP typing scheme is based methods described in _[Li et. al.](https://journals.asm.org/doi/full/10.1128/mBio.00756-16)_ (citation below).

### Conda

`pbptyper`is available from Bioconda

```{bash}
mamba create -n pbptyper -c conda-forge -c bioconda pbptyper
conda activate pbptyper
pbptyper --help
```

## Usage

```{bash}
 Usage: pbptyper [OPTIONS]

 In silico Penicillin Binding Protein (PBP) typer for Streptococcus pneumoniae assemblies

╭─ Options ───────────────────────────────────────────────────────────────────────────────────────────╮
│    --version                  Show the version and exit.                                            │
│ *  --assembly        TEXT     Input assembly in FASTA format (gzip is OK) [required]                │
│    --db              TEXT     Input database in uncompressed FASTA format [default: db/]            │
│    --prefix          TEXT     Prefix to use for output files [default: basename of input]           │
│    --outdir          TEXT     Directory to save output files [default: ./]                          |
│    --min_pident      INTEGER  Minimum percent identity to count a hit [default: 95]                 │
│    --min_coverage    INTEGER  Minimum percent coverage to count a hit [default: 95]                 │
│    --min_ani         INTEGER  Minimum S. pneumoniae ANI to predict PBP Type [default: 95]           │
│    --check                    Check dependencies are installed, then exit                           │
│    --quiet                    Suppress all output                                                   │
│    --help                     Show this message and exit.                                           │
╰─────────────────────────────────────────────────────────────────────────────────────────────────────╯
```

### --assembly

An assembly in FASTA format (compressed with gzip, or uncompressed) to predict the PBP type on.

### --db

A path to a directory containing FASTA files for 1A, 2B, and 2X proteins. In most cases using the
default value will be all that is needed.

### --prefix

The prefix to use for the output files. If a prefix is not given, the basename
of the input assembly will be used.

### --outdir

The directory to save result files. If the directory does not exist is will be created.

### --min_pident

The minimum percent identity for a blast hit to be considered for typing. The is compared
against the `pident` column of the blast output.

### --min_coverage

The minimum coverage of a PBP to be considered for typing. The is compared
against the `qcovs` column of the blast output.

### --min_ani

The minimum _S. pneumoniae_ ANI required to predict PBP type. The ANI is calculated using
[FastANI](https://github.com/ParBLiSS/FastANI) and the
[GCF_000006885](https://www.ncbi.nlm.nih.gov/assembly/GCF_000006885/) reference genome.

### --check

Check that the `fastANI` and `tblastn` dependencies are available from the PATH.

### --quiet

This will quiet STDOUT quite a bit.

## Output Files

| Filename                  | Description                                       |
|---------------------------|---------------------------------------------------|
| `{PREFIX}.tsv`            | A tab-delimited file with the predicted PBP type  |
| `{PREFIX}.fastani.tsv`    | A tab-delimited file of fastANI results           |
| `{PREFIX}-1A.tblastn.tsv` | A tab-delimited file of all blast hits against 1A |
| `{PREFIX}-2B.tblastn.tsv` | A tab-delimited file of all blast hits against 2B |
| `{PREFIX}-2X.tblastn.tsv` | A tab-delimited file of all blast hits against 2X |

### Example `{PREFIX}.tsv`

This file will contain the final predicted PBP type based on highest coverage, percent identity, and bitscore
(_ties are broken in that order_). Here's what to expect the output to look like:

```{bash}
sample	pbptype	ani	1A_coverage	1A_pident	1A_bitscore	2B_coverage	2B_pident	2B_bitscore	2X_coverage	2X_pident	2X_bitscore	comment
SRR2912551	23:0:2	98.70	100	100.000	556	100	100.000	567	100	100.000	741	
```

| Column Name | Description                                                    |
|-------------|----------------------------------------------------------------|
| sample      | Name of the sample processed                                   |
| pbptype     | The predicted PBP type (1A:2B:2X)                              |
| ani         | ANI against a _S. pneumoniae_ (GCF_000006885) reference genome |
| 1A_coverage | The percent coverage of the top hit against the 1A PBP protein |
| 1A_pident   | The percent identity of the top hit against the 1A PBP protein |
| 1A_bitscore | The bitscore of the top hit against the 1A PBP protein         |
| 2B_coverage | The percent coverage of the top hit against the 2B PBP protein |
| 2B_pident   | The percent identity of the top hit against the 2B PBP protein |
| 2B_bitscore | The bitscore of the top hit against the 2B PBP protein         |
| 2X_coverage | The percent coverage of the top hit against the 2X PBP protein |
| 2X_pident   | The percent identity of the top hit against the 2X PBP protein |
| 2X_bitscore | The bitscore of the top hit against the 2X PBP protein         |
| comment     | Any comments associated with the analysis                      |

#### PBPType Interpretation

In the example above `pbptyper` predicted the PBP Type to be _23:0:2_. This means for 1A, allele 23 had
a perfect match, for 2B, allele 0 had a perfect match, and finally for 2X, allele 2 had a perfect
match.

Here's a break down of possible ID values:

| Allele ID       | Description                                                                                        |
|-----------------|----------------------------------------------------------------------------------------------------|
| 0+ (any number) | A perfect match was found against an existing allele ID (_yes the count starts at 0!_)             |
| MULTIPLE        | There was a perfect match against multiple allele IDs for a loci, and a ID could not be determined |
| NEW             | A hit was made that was not perfect but exceeded the `min_pident` and `min_coverage` thresholds    |
| NA              | No hits exceeded the `min_pident` and `min_coverage` thresholds                                    |
| NOTSPN          | Input assembly did not exceed `min_ani` threshold against _S. pneumoniae_                          |

#### Merging Multiple Runs

You can a tool like [csvtk - concat](https://bioinf.shenwei.me/csvtk/usage/#concat) from [Wei Shen](https://github.com/shenwei356)
to easily merge the results from multiple samples. Here's an example of how to do so:

```{bash}
csvtk concat --tabs --out-tabs --out-file pbptyper.tsv $(ls *.tsv | grep -v "tblastn")
```

This is just one way to do it, there are many other ways.

### Example `{PREFIX}-{1A|2B|2X}.tblastn.tsv`

The blast results include sequences which will not display very nicely here. But, don't fret,
it is the standard `-outfmt 6`, so tab-delimited and easy to parse.

## Naming

The original implementation was called _PBP-Gene_Typer.pl_. For this rewrite in Python, I just shortened it to
`pbptyper`. _Haha not much else to the naming!_

## Citation

If you use this tool please cite the following:

**PBP Typing Method**  
_Li Y, Metcalf BJ, Chochua S, et al. [Penicillin-binding protein transpeptidase signatures for tracking and predicting β-lactam resistance levels in Streptococcus pneumoniae](https://journals.asm.org/doi/full/10.1128/mBio.00756-16) mBio 7(3):60 pii:e00756–16. (2016)_  

**pbptyper**  
_Petit III RA [pbptyper: In silico Penicillin Binding Protein (PBP) typer for _Streptococcus pneumoniae_ assemblies](https://github.com/rpetit3/pbptyper) (GitHub)_  

**BLAST+**  
_Camacho C, Coulouris G, Avagyan V, Ma N, Papadopoulos J, Bealer K, Madden TL [BLAST+: architecture and applications.](http://dx.doi.org/10.1186/1471-2105-10-421) BMC Bioinformatics 10, 421 (2009)_  

**fastANI**  
_Jain C, Rodriguez-R LM, Phillippy AM, Konstantinidis KT, Aluru S [High throughput ANI analysis of 90K prokaryotic genomes reveals clear species boundaries.](http://dx.doi.org/10.1038/s41467-018-07641-9) Nat. Commun. 9, 5114 (2018)_  
