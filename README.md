# SILVA README #

SilVA: Silent variant analysis using random forests.
*"When a synonymous mutation falls in the exome, does it make a sound?"*


SilVA is a tool for the automated harmfulness prediction of **SYNONYMOUS** single-nucleotide variants. Given variants in a VCF file, SilVA will rank the rare synonymous variants according to their predicted harmfulness. SilVA predicts the harmfulness of mutations using features that include conservation, codon usage, splice sites, splicing enhancers and suppressors, and mRNA folding free energy.

If you use this tool, please cite:

    Buske OJ, Manickaraj A, Mital S, Ray PN, Brudno M. (2013)
    Identification of deleterious synonymous variants in human genomes.
    Bioinformatics, doi:10.1093/bioinformatics/btt308

Send questions, comments, and difficulties to: silva-snv@cs.toronto.edu


## Quickstart ##

1. Download SilVA:

  ```bash
wget http://compbio.cs.toronto.edu/silva/release/silva-1.1.1.tar.gz
tar -xzf silva-1.1.1.tar.gz
cd silva-1.1.1
```

2. Install dependencies:

  ```bash
./setup.sh
```

3. Preprocess VCF file:

  ```bash
    ./silva-preprocess OUTDIR VCF
```

4. Run models and print highest-scoring synonymous variants:

  ```bash
    ./silva-run OUTDIR | head
```

## Overview ##

### Prerequisites ###

- Linux or Mac OS, x86
- 6GB of RAM
- Python 2.6 or 2.7 in your PATH
  - The 'numpy' Python package
- Perl in your PATH (for maxentscan)
- R in your PATH (for randomForest)

### Dependencies ###

SilVA requires several tools and databases to run. Most of these were included with this release, but the `setup.sh` script will download and configure the rest.

- included: maxentscan, available from:
  http://genes.mit.edu/burgelab/maxent/download/
- included: twobitreader, available on pypi:
  https://pypi.python.org/pypi/twobitreader
- setup.sh: randomForest package, available on CRAN:
  http://cran.r-project.org/web/packages/randomForest/
- setup.sh: UNAfold, available from:
  http://mfold.rna.albany.edu/?q=DINAMelt/software
- setup.sh: ViennaRNA, available from:
  http://www.tbi.univie.ac.at/~ronny/RNA/
- setup.sh: SilVA databases, available from:
  http://compbio.cs.toronto.edu/silva/release/silva-1.1.1_data.tar.gz
- setup.sh: hg19.2bit, available from:
  http://hgdownload.cse.ucsc.edu/goldenPath/hg19/bigZips/hg19.2bit

**Note**: _if you have the SilVA databases already installed, you can point SilVA to them by changing the appropriate lines in the `init.sh` file._

### Data files ###

After running setup.sh, all necessary data files should be automatically downloaded into the data/ directory. Here is a list of the files that should be there:
- `refGene.ucsc.gz`
- `1000gp.refGene.vcf.gz`
- `gerp.refGene.table.gz`
- `hg19.2bit`

**Note**: _The very first time you run SilVA, it will take much longer than normal (~45min longer) because the reference genome needs to be processed and mapped to the refSeq gene annotations and data files need to be parsed. This only needs to be done once, since the following pre-processed databases are saved to the data/ directory for future runs:_
- `refGene.pkl`
- `1000gp.refGene.pkl`
- `gerp.refGene.pkl`

### Input File Format ###

Single-nucleotide variants should be given to SilVA in VCF or a VCF-like file format. This file may be gzip'd, but then must have a '.gz' extension. There should be one line per variant, with tab-delimited fields:

1. `chrom` - chromosome (any 'chr' prefix will be trimmed)
2. `pos` - the position of the variant, 1-indexed
3. `id` - the id field is ignored, but is here to provide compatibility with the VCF format.
4. `ref` - the reference nucleotide
5. `alt` - the alternate nucleotide (if multiple are present, comma-separated, only the first will be used)
... Additional columns can be present and are ignored

### Output Format ###

SilVA will print the synonymous variants to stdout, ordered by score, with the variants most likely to be harmful listed first. Variants are first filtered down to just rare and novel synonymous variants. The variants are then annotated with a number of features and the machine learning model is used to rank the variants. The output contains the following tab-delimited columns:

1. The variant rank, out of all synonymous variants considered.
2. The SilVA score, between 0 and 1, where close to 1 is more likely to be harmful.
3. The variant classification, based upon several SilVA score thresholds.
3. HGNC gene name
4. RefSeq mRNA identifier
... Fields from the input file


## Installation ##

SilVA is packaged with most of its dependencies. The remaining few can be downloaded and configured by running the 'setup.sh' script from the root directory of this package:

1. Download, untar, and unzip the package tarball:

  ```bash
wget http://compbio.cs.toronto.edu/silva/release/silva-1.1.1.tar.gz
tar -xzf silva-1.1.1.tar.gz
cd silva-1.1.1
```

2. Run the setup script in the package's root directory:

  ```bash
./setup.sh
```

**Note**: _SilVA uses a number of environment variables to communicate important paths and parameters, such as what directory to use for temporary files and what allele frequency threshold to use. SilVA has default settings that should work, but will defer to any settings in your environment (so you can hard-code values by exporting variables in your `~/.bashrc`, for example). To see a list of these variables or to change their settings, see the `init.sh` script._


## Running SilVA ##

In addition to running SilVA on your local machine, we provide a convenience script to dispatch multiple SilVA runs across an SGE cluster. SilVA uses TMPDIR for all temporary files. If TMPDIR is not set, it will use the current directory.

### Local machine ###

To run SilVA on your local machine, you should have already installed dependencies with the 'setup.sh' script. You can then run SilVA on your file of variants, <VCF>, (see Input Format) from the root directory of this package (or add this directory to your PATH):

1. Filter an annotate the variants in the VCF file (this takes a while):

  ```bash
./silva-preprocess <OUTDIR> <VCF>
```

2. Run the trained models on the annotated variants and print the top variants in order of decreasing predicted harmfulness (this is fast):

  ```bash
./silva-run <OUTDIR> | head
```

SilVA should handle errors and early termination gracefully, so if something goes wrong, you can re-run the same command and it will pick up where it left off.

### Example ###

The 'example/' folder in the root directory of this package contains a sample input file for you to test SilVA on. You can run SilVA on it with the following commands:

```bash
./silva-preprocess example example/example.vcf
./silva-run example | head
```

You should see the known-harmful ACVRL1 variant appear at the top! Granted, for simplicity, the example only has 28 other rare synonymous variants in it. SilVA's expected output on this example is included in 'example/example.out'.

### SGE Cluster ###

In case you have a large number of VCF files and an SGE cluster, we have provided a dispatch script for your convenience, 'sge/dispatch.sh'. Because SGE configurations vary widely, you should first open the file and take a look at the three SGE configuration variables at the top of the script. Edit them as you see fit. You can then dispatch SGE jobs for a number of VCF files from the root directory of the package with:

```bash
./sge/dispatch.sh <OUTDIR> <VCF1> <VCF2> ...
```

## Support ##

Send questions, comments, and difficulties to: `silva-snv@cs.toronto.edu`
