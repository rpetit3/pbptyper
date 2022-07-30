# pbptyper
In silico Penicillin Binding Protein (PBP) typer for _Streptococcus pneumoniae_ assemblies

## A Quick Note

The original [CDC StrepLab](https://www.cdc.gov/streplab/pneumococcus/mic.html) implementation is
available at [BenJamesMetcalf/Spn_Scripts_Reference](https://github.com/BenJamesMetcalf/Spn_Scripts_Reference) and
[BenJamesMetcalf/JanOw_Dependencies](https://github.com/BenJamesMetcalf/JanOw_Dependencies).

I'll be the first to admit, my Perl skills aren't what they used to be. I tried. But I also wanted to
simplify things a little. The original method takes FASTQs, trims adapters, assembles with Velvet, 
predicts genes, Blast genes, etc... I think these days its much easier to just start with assemblies. 
I changed things so that, all that's needed is an assembly and `tblastn` is used to blast the PBP
proteins against the assembly.

## Introduction

`pbptyper` is a tool to identify the Penicillin Binding Protein (PBP) of _Streptococcus pneumoniae_ assemblies. 
Using an input assembly (uncompressed or gzip-compressed), the PBP 1A, 2B, and 2X proteins are blasted against
the assembly from which a PBP type is predicted.

The PBP typing scheme is based methods described in _[Li et. al.](https://journals.asm.org/doi/full/10.1128/mBio.00756-16)_ (citation below).

### Conda

Once released this will be added to Bioconda, in the meantime this should work.

```{bash}
mamba create -n pbptyper-dev -c conda-forge -c bioconda rich-click executor 'python>=3.7' blast
conda activate pbptyper-dev
git clone git@github.com:rpetit3/pbptyper.git
cd pbptyper
bin/pbptyper --help
```

## Usage

```{bash}
 Usage: pbptyper [OPTIONS]

 In silico Penicillin Binding Protein (PBP) typer for Streptococcus pneumoniae assemblies

╭─ Options ───────────────────────────────────────────────────────────────────────────────────────────╮
│    --version                  Show the version and exit.                                            │
│ *  --assembly        TEXT     Input assembly in FASTA format (gzip is OK) [required]                │
│    --db              TEXT     Input database in uncompressed FASTA format [default: db/] │
│    --prefix          TEXT     Prefix to use for output files [default: basename of input]           │
│    --outdir          TEXT     Directory to save output files [default: ./]                          |
│    --min_pident      INTEGER  Minimum percent identity to count a hit [default: 95]                 │
│    --min_coverage    INTEGER  Minimum percent coverage to count a hit [default: 95]                 │
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

## Output Files

| Filename                  | Description                                       |
|---------------------------|---------------------------------------------------|
| `{PREFIX}.tsv`            | A tab-delimited file with the predicted PBP type  |
| `{PREFIX}-1A.tblastn.tsv` | A tab-delimited file of all blast hits against 1A |
| `{PREFIX}-2B.tblastn.tsv` | A tab-delimited file of all blast hits against 2B |
| `{PREFIX}-2X.tblastn.tsv` | A tab-delimited file of all blast hits against 2X |

### Example `{PREFIX}.tsv`

This file will contain the final predicted PBP type based on highest coverage, percent identity, and bitscore
(_ties are broken in that order_). Here's what to expect the output to look like:

```{bash}
sample  pbptype 1A_coverage     1A_pident       2B_coverage     2B_pident       2X_coverage     2X_pident
SRR2912551.fna  23:0:2  100     100.000 100     100.000 100     100.000
```

| Column Name | Description                                |
|-------------|--------------------------------------------|
| sample      | Name of the sample processed               |
| pbptype     | The predicted PBP type (1A:2B:2X)          |
| 1A_coverage | The percent coverage of the 1A PBP protein |
| 1A_pident   | The percent identity of the 1A PBP protein |
| 2B_coverage | The percent coverage of the 2B PBP protein |
| 2B_pident   | The percent identity of the 2B PBP protein |
| 2X_coverage | The percent coverage of the 2X PBP protein |
| 2X_pident   | The percent identity of the 2X PBP protein |

### Example `{PREFIX}-{1A|2B|2X}.tblastn.tsv`

The blast results include sequences which won't display very nicely here. But, don't fret,
it is the standard `-outfmt 6`, so tab-delimited and easy to parse.

## Naming

The original implementation was called _PBP-Gene_Typer.pl_. For this rewrite in Python, I just shortened it to
`pbptyper`. _Haha not much else to the naming!_

## Citation

If you use this tool please cite the following:

**PBP Typing Method**  
_Li Y, Metcalf BJ, Chochua S, et al. [Penicillin-binding protein transpeptidase signatures for tracking and predicting β-lactam resistance levels in Streptococcus pneumoniae](https://journals.asm.org/doi/full/10.1128/mBio.00756-16) mBio 7(3):60 pii:e00756–16. (2016)_  

**pasty**  
_Petit III RA [pasty: A tool easily taken advantage of for in silico serogrouping of Pseudomonas aeruginosa isolates](https://github.com/rpetit3/pasty) (GitHub)_  

**BLAST+**  
_Camacho C, Coulouris G, Avagyan V, Ma N, Papadopoulos J, Bealer K, Madden TL [BLAST+: architecture and applications.](http://dx.doi.org/10.1186/1471-2105-10-421) BMC Bioinformatics 10, 421 (2009)_  
