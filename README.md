<img alt="logo" src="https://github.com/COMBINE-lab/alevin-fry/raw/master/docs/logo.png" width="200">

# alevin-fry ![Rust](https://github.com/COMBINE-lab/alevin-fry/workflows/Rust/badge.svg) [![Anaconda-Server Badge](https://anaconda.org/bioconda/alevin-fry/badges/installer/conda.svg)](https://conda.anaconda.org/bioconda) [![Anaconda-Server Badge](https://anaconda.org/bioconda/alevin-fry/badges/platforms.svg)](https://anaconda.org/bioconda/alevin-fry) [![Anaconda-Server Badge](https://anaconda.org/bioconda/alevin-fry/badges/license.svg)](https://anaconda.org/bioconda/alevin-fry) ![GitHub tag (latest SemVer)](https://img.shields.io/github/v/tag/combine-lab/alevin-fry?style=flat-square)

`alevin-fry` is a suite of tools for the rapid, accurate and memory-frugal processing single-cell and single-nucleus sequencing data.  It consumes RAD files generated by `salmon alevin`, and performs common operations like generating permit lists, and estimating the number of distinct molecules from each gene within each cell.  The focus in `alevin-fry` is on safety, accuracy and efficiency (in terms of both time and memory usage).

You can read the paper describing alevin fry, "Alevin-fry unlocks rapid, accurate, and memory-frugal quantification of single-cell RNA-seq data" [here](https://www.nature.com/articles/s41592-022-01408-3), and the pre-print [on bioRxiv](https://www.biorxiv.org/content/10.1101/2021.06.29.450377v1).

* [**Quickstart guide with a unified singularity container**](https://github.com/COMBINE-lab/alevin-fry#a-quick-start-run-through-on-sample-data)

* **Relationship to alevin**: Alevin-fry has been designed as the successor to alevin. It subsumes the core features of alevin, while also providing important new capabilities and considerably improving the performance profile. We anticipate that new method development and feature additions will take place primarily within the alevin-fry codebase.  Thus, we encourage users of alevin to migrate to alevin-fry when feasible.  That being said, alevin is still actively-maintained and supported, so if you are using it and not ready to migrate you can continue to ask questions and post issues in [the salmon repository](https://github.com/COMBINE-lab/salmon).

## Documentation 

Alevin-fry is under active development.  However, you can find the documentation on [read the docs](https://alevin-fry.readthedocs.io/en/latest/).  We try to keep the documentation up to date with the latest developments in the software.  Additionally, there is a series of tutorial for using alevin-fry for processing different types of data that you can find [here](https://combine-lab.github.io/alevin-fry-tutorials/).

## FAQs 

Are you curious about processing details like [whether to use a sparse or dense index](https://github.com/COMBINE-lab/alevin-fry/discussions/38)? Do you have a question that isn't necessarily a bug report or feature request, and that isn't readily answered by the documentation or tutorials?  Then please feel free to ask over in the [Q&A](https://github.com/COMBINE-lab/alevin-fry/discussions/categories/q-a).

## Sister repositories

The generation of the reduced alignment data (RAD) files processed by alevin-fry is done by [salmon](https://github.com/COMBINE-lab/salmon). The latest version of salmon is available [on GitHub](https://github.com/COMBINE-lab/salmon/releases), via [bioconda](https://bioconda.github.io/recipes/salmon/README.html), and on [dockerhub](https://hub.docker.com/layers/combinelab/salmon/latest/images/sha256-f86324c6aeacb627e3c589562ab9e2564a6d51a3892a697669d3f23d0b9d81a8?context=explore). 

The [`usefulaf`](https://github.com/COMBINE-lab/usefulaf) repository contains scripts in functions that are useful in helping to prepare input for alevin-fry processing, importing alevin-fry output into downstream analysis evnironemnts, and even [running common configurations of alevin-fry more simply](https://github.com/COMBINE-lab/usefulaf/blob/main/bash/simpleaf.sh).  This repository also contains the relevant [Python function](https://github.com/COMBINE-lab/usefulaf/blob/main/python/load_fry.py) for loading fry output (specifically in USA mode) in a convenient way into [scanpy](https://scanpy.readthedocs.io/en/stable/) (i.e. as [AnnData](https://scanpy.readthedocs.io/en/latest/usage-principles.html#anndata) objects) for subsequent Python-based processing in scanpy.

The [`roe`](https://github.com/COMBINE-lab/roe) and [`pyroe`](https://github.com/COMBINE-lab/pyroe) repositories provide tools to help easily construct a _splici_ transcriptome from a reference genome and GTF file in `R` and `python` respectively.

The [`fishpond`](https://github.com/mikelove/fishpond) package — maintained by @mikelove and his lab — contains the recommended relevant functions for reading `alevin-fry` output (particularly USA-mode output) into the R ecosystem, in the form of a [`singleCellExperiment`](https://bioconductor.org/packages/release/bioc/html/SingleCellExperiment.html) object.

The [`alevinqc`](https://github.com/csoneson/alevinQC) package — maintained by @csoneson — provides tool and functions for performing quality control and assessment downstream of `alevin-fry`.

## Installing from bioconda


Alevin-fry is available for both x86 linux and OSX platforms [using bioconda](https://anaconda.org/bioconda/alevin-fry).

With `bioconda` in the appropriate place in your channel list, you should simply be able to install via:


```{bash}
$ conda install alevin-fry
``` 

## Installing from crates.io

Alevin-fry can also be installed from [`crates.io`](https://crates.io/crates/alevin-fry) using `cargo`.  This can be done with the following command:

```{bash}
$ cargo install alevin-fry
```

## Building from source

If you want to use features or fixes that may only be available in the latest develop branch (or want to build for a different 
architecture), then you have to build from source.  Luckily, `cargo` makes that easy; see below.

Alevin-fry is built and tested with the latest (major & minor) stable version of [Rust](https://www.rust-lang.org/). While it will likely compile fine with slightly older versions of Rust, this is not a guarantee and is not a support priority.  Unlike with C++, Rust has a frequent and stable release cadence, is designed to be installed and updated from user space, and is easy to keep up to date with [rustup](https://rustup.rs/). Thanks to cargo, building should be as easy as:

```{bash}
$ cargo build --release
```

subsequent commands below will assume that the executable is in your path.  Temporarily, this can 
be done (in bash-like shells) using:

```{bash}
$ export PATH=`pwd`/target/release/:$PATH
```

## A quick start run through on sample data

Here, we show how to perform a complete analysis on the [1k PBMCs from a Healthy Donor data from 10X Genomics](https://www.10xgenomics.com/resources/datasets/1-k-pbm-cs-from-a-healthy-donor-v-3-chemistry-3-standard).  This run through includes **all steps**, even extracting the _splici_ sequence and building the salmon index, which you typically would not do _per-sample_.  To make this sample as easy as possible to follow, we have bundled all of the required software and utilities in a singularity container that we use in the commands below.

### Download input data and singularity container

First, create a working directory with sufficient space to download all of the input data and to hold the output (50GB should be sufficient).  We alias this directory and use the alias below so that you can easily set it to something else if you want and still copy and paste the later commands.

```{bash}

$ mkdir af_test_workdir
$ export AF_SAMPLE_DIR=$PWD/af_test_workdir
```

Then, we download all of our test data.  This consist of the human reference genome and annotation (based, in this case, on the [CellRanger](https://support.10xgenomics.com/single-cell-gene-expression/software/overview/welcome) 3.0 reference annotation) and the FASTQ files from the [PBMC1k (v3) healthy donor samples](https://www.10xgenomics.com/resources/datasets/1-k-pbm-cs-from-a-healthy-donor-v-3-chemistry-3-standard).

```{bash}
$ cd $AF_SAMPLE_DIR
$ mkdir -p human_CR_3.0/fasta
$ mkdir -p human_CR_3.0/genes
$ wget -v -O human_CR_3.0/fasta/genome.fa -L https://umd.box.com/shared/static/3kuh1lc03bxg1d3hi1jfloez7zoutfjc
$ wget -v -O human_CR_3.0/genes/genes.gtf -L https://umd.box.com/shared/static/tvyg43710ufuuvp8mnuoanowm6xmkbjk
$ mkdir -p data/pbmc_1k_v3_fastqs
$ wget -v -O data/pbmc_1k_v3_fastqs/pbmc_1k_v3_S1_L001_R2_001.fastq.gz -L https://umd.box.com/shared/static/bmhtt9db8ojhmbkb6d98mt7fnsdhsymm
$ wget -v -O data/pbmc_1k_v3_fastqs/pbmc_1k_v3_S1_L002_R2_001.fastq.gz -L https://umd.box.com/shared/static/h8ymvs2njqiygfsu50jla2uce6p6etke
$ wget -v -O data/pbmc_1k_v3_fastqs/pbmc_1k_v3_S1_L001_R1_001.fastq.gz -L https://umd.box.com/shared/static/hi8mkx1yltmhnl9kn22n96xtic2wqm5i
$ wget -v -O data/pbmc_1k_v3_fastqs/pbmc_1k_v3_S1_L002_R1_001.fastq.gz -L https://umd.box.com/shared/static/4sn4pio63kk7pku52eo3xg9ztf5tq1ul
```

Finally, we'll download the [singularity](https://sylabs.io/singularity/) image that contains all of the software we'll need to do our processing.

```{bash}
$ wget -v -O usefulaf.sif https://umd.box.com/shared/static/bcd8io9fbjc321pfgcomues5oe2a12cz
```

or, alternatively, you can pull the docker image directly from Dockerhub and have singularity convert it for you

```{bash}
$ singularity pull docker://combinelab/usefulaf:latest
```

### Info about the singularity container

The singularity container we just downloaded above contains a recent release of `salmon` (v1.8.0) and `alevin-fry` (v0.5.0), as well as an installation of `R` and all of the packages needed to build the _splici_ index.

To build the reference index (and quantify) we'll use the [simpleaf](https://github.com/COMBINE-lab/usefulaf/blob/main/bash/simpleaf) wrapper.  This is a shell script written around `salmon`, `alevin-fry`, and the _splici_ index construction code that simplifies processing by grouping together related commands, using a fixed directory structure for processing, and also by eliminating some different options that are otherwise exposed by `salmon` and `alevin-fry` (e.g. it builds the `sparse` index, maps in `sketch` mode etc.).  If you would like to run the "raw" commands, the Singularity image contains `salmon` and `alevin-fry` in the path, and the `R` script to construct the _splici_ index at `/usefulaf/R/build_splici_ref.R`, so you can explore more detailed options.

### Building the splici reference and index

To build our reference index (this will both extract the _splici_ fasta and transcript to gene mapping, and build the `salmon` index on it), use the following command (this should generally take ~1hr or less):


```{bash}
$ singularity exec --cleanenv \
--bind $AF_SAMPLE_DIR:/workdir \
--pwd /usefulaf/bash usefulaf.sif \
./simpleaf index \
-f /workdir/human_CR_3.0/fasta/genome.fa \
-g /workdir/human_CR_3.0/genes/genes.gtf \
-l 91 -t 16 -o /workdir/human_CR_3.0_splici
```

### Quantifying the sample

Given the constructed index (which will be written by the above command to `$AF_SAMPLE_DIR/human_CR_3.0_splici/index`), the next step is to quantify the sample against this index.  This can be done with the following command (this should generally take only a few minutes), which will run `salmon` to generate the RAD file in `sketch` mode, perform _unfiltered_ permit-list generation --- automatically downloading the appropriate external permit-list --- collate the RAD file and quantify the gene counts using the `cr-like` strategy):

```{bash}
$ singularity exec --cleanenv \
--bind $AF_SAMPLE_DIR:/workdir \
--pwd /usefulaf/bash usefulaf.sif \
./simpleaf quant \
-1 /workdir/data/pbmc_1k_v3_fastqs/pbmc_1k_v3_S1_L001_R1_001.fastq.gz,/workdir/data/pbmc_1k_v3_fastqs/pbmc_1k_v3_S1_L002_R1_001.fastq.gz \
-2 /workdir/data/pbmc_1k_v3_fastqs/pbmc_1k_v3_S1_L001_R2_001.fastq.gz,/workdir/data/pbmc_1k_v3_fastqs/pbmc_1k_v3_S1_L002_R2_001.fastq.gz \
-i /workdir/human_CR_3.0_splici/index \
-o /workdir/quants/pbmc1k_v3 \
-f u -c v3 -r cr-like \
-m /workdir/human_CR_3.0_splici/ref/transcriptome_splici_fl86_t2g_3col.tsv \
-t 16
```

### Output

At the end of this process, the directory `$AF_SAMPLE_DIR/quants/pbmc1k_v3/quant` will have the final output of running `alevin-fry`'s `quant` command.  The `alevin` subdirectory will include a file specifying the row names (cell barcode), the column names (unspliced, spliced and ambiguous genes) and the counts (in MTX coordinate format).  You can load these counts up in your favorite analysis environment to explore further.

**R** : In [R](https://www.r-project.org/), you can make use of the `R` [`load_fry()`](https://github.com/COMBINE-lab/usefulaf/blob/main/R/load_fry.R) function here, and read the input with the command:

```{R}
m <- load_fry("$AF_SAMPLE_DIR/quants/pbmc1k_v3/quant")
```

where `$AF_SAMPLE_DIR` is appropriately replaced by the path to the working directory we chose at the start of this exercise.  This will return a [SingleCellExperiment](https://bioconductor.org/packages/release/bioc/html/SingleCellExperiment.html) object containing the counts for this experiment.  The stand-alone `load_fry()` function is part of [`fishpond`](https://bioconductor.org/packages/release/bioc/html/fishpond.html), and the function is documented in detail [here](https://mikelove.github.io/fishpond/reference/loadFry.html).


**Python** : In [python](https://www.python.org/), you can make use of the `python` [`load_fry()`](https://github.com/COMBINE-lab/usefulaf/blob/main/python/load_fry.py) function, which relies on [scanpy](https://scanpy.readthedocs.io/en/stable/).  To read the input you can use the following command:

```{python}
m = load_fry("$AF_SAMPLE_DIR/quants/pbmc1k_v3/quant")
```

where, again `$AF_SAMPLE_DIR` is appropriately replaced by the path to the working directory we chose at the start of this exercise.  This will return a `scanpy` [`AnnData`](https://anndata.readthedocs.io/en/latest/) object with the counts.

From here, you can use your favorite downstream analysis packages (like [`Seurat` in R](https://satijalab.org/seurat/) or [`scanpy` in python](https://scanpy.readthedocs.io/en/stable/)) to perform quality control, filtering and analysis of your data.

## A note about preparing a _splici_ (spliced + intron) reference

In [the manuscript describing alevin-fry](https://www.biorxiv.org/content/10.1101/2021.06.29.450377v1), we primarily make use of an index that is built over spliced + intron sequence, which we refer to as a _splici_ reference.  This is also what we build in the quick start example above. To make the construction of the relevant reference sequence (and the 3 column TSV file you will need for Unspliced/Spliced/Ambiguous (USA) quantification) simple, we have written an R script that will process a genome and GTF file and produce the splici reference which you can then index with [`salmon`](https://github.com/COMBINE-lab/salmon) as normal.

First, checkout the [`usefulaf`](https://github.com/COMBINE-lab/usefulaf) repository and navigate to the `R` directory.  Then, we'll run the 
`build_splici_ref.R` script.

```
$ ./build_splici_ref.R <path_to_genome_fasta> <path_to_gtf> <target_read_length> <output_dir>
```

where `$` indicated your command prompt. In addition to these required positional arguments, there are a few optional arguments that you can find by running 

```
$ ./build_splici_ref.R -h
```

After you have run this script, your output directory should contain 3 files:

```
<output_dir>/transcriptome_splici_fl<target_read_length-5>.fa
<output_dir>/transcriptome_splici_fl<target_read_length-5>_t2g.tsv
<output_dir>/transcriptome_splici_fl<target_read_length-5>_t2g_3col.tsv
```

The first file contains the _splici_ reference sequence that you should index with `salmon`, and the third contains the 3-column transcript-to-gene mapping 
that you should pass to `alevin-fry` during the `quant` phase.

If you have any questions about preparing the splici reference, or otherwise about processing your data with `alevin-fry` please feel free to open an issue 
here on GitHub!

## Citing alevin-fry

If you use `alevin-fry` in your work, please cite:

```
He, D., Zakeri, M., Sarkar, H. et al. Alevin-fry unlocks rapid, accurate and memory-frugal quantification of single-cell RNA-seq data. Nat Methods 19, 316–322 (2022). https://doi.org/10.1038/s41592-022-01408-3
```

**BibTeX:**
```
@Article{He2022,
author={He, Dongze and Zakeri, Mohsen and Sarkar, Hirak and Soneson, Charlotte and Srivastava, Avi and Patro, Rob},
title={Alevin-fry unlocks rapid, accurate and memory-frugal quantification of single-cell RNA-seq data},
journal={Nature Methods},
year={2022},
month={Mar},
day={01},
volume={19},
number={3},
pages={316-322},
issn={1548-7105},
doi={10.1038/s41592-022-01408-3},
url={https://doi.org/10.1038/s41592-022-01408-3}
}
```

