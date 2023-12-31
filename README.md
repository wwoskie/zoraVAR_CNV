# IBRE Biohack zoraVAR_CNV

This project is inspired by IBRE Biohack hackaton problem on CNV DAta analisys.

Gains and losses of DNA are prevalent in cancer and emerge as a consequence of inter-related processes of replication stress, mitotic errors, spindle multipolarity and breakage–fusion–bridge cycles, among others, which may lead to chromosomal instability and aneuploidy.

## Causes:
- non-homologous end joining;
- template switching;
- chromosome breakage during mitosis -> chromothripis;
- replication timing:
- early replication -> unmatched DNA regions can be mistakenly combined;
- late replication -> nonhomologous recombination;
- specific chromatin remodeling events

## Two major ways for CNA calling

### BAF (B-allele frequency)

“B” allele is the non-reference allele observed in a germline heterozygous SNP, i.e. in the normal/control sample. Estimation from a SNP b-allele frequencies works by comparing the shift in allele frequency of heterozygous, germline SNPs in the tumor sample from the expected ~50%.

`BAF = coverage Alt / (coverage Alt + coverage Ref)`

### Depth ratio

Coverage of certain area in tumor divided by coverage of same area in control provides info about change in copy number of this area. In real life this ratio is also needs to be corrected with GC-content correction, purity/ploidy correction and some other corrections.

## Usage

This instrument can be used in two different modes:

- python-notebook format
- as a library via `import` command

Both instruments require the following packages:

- `pysam`
- `pandas`
- `matplotlib`
- `numpy`
- `seaborn`

We strongly suggest installing these packages via `conda install <package_name>` command. You can install conda following guide on Anaconda official site [this](https://www.anaconda.com/).

### Input data

This tool uses the following data for input:
- patient's normal tissue WES `.bam`-file
- patients tumor tissue WES `.bam`-file
- [*optionally*]  `.bam.bai` index files for both tumor and normal, the tool though can generate them automatically if not present
- ['CNV-kit'](https://cnvkit.readthedocs.io/en/stable/) output
- FACETS output

### Tool structure

This tool consists of three parts:
- FEM focal events module
- BAF module
- CNV post-processing module

#### Focal events module

This module comprises three main functions:
- `gene_extractor` - used to process input `.bam` files
- `binner` - to create data bins for future analisys
- `main` function to handle input data, process intermediates and manage the above two fuctions

This module takes `.bed`-like file in `.tsv` format with gene names and coordinates and creates dataframe from it. Then it accepts '.bam` files from tumor and normal tissue. After processing it creates `.bam` file subset of data plus border shift (can be passed as a parameter). After that it calculates bins, rolling median, depth-ratio and copy number data along with median rolling depth-ratio and binning graph output.

## Pulling data from cnvKit output
We recomend to use `pull_cnv.ipynb` for pulling of segments of interest.  
*Comand line implementation is under development*

Arguments are assigned with keyborf inpu.

Arguments:
- path to .cns outpu of cnvKit.
- path to .bed with genes of interest.

Script plots log2 depth ratios for given area and prints dictionary with called copynumbers for corresponding genes.

## Calculation gene copy number using BAF values

Python script [BAF_calc.py](./BAF_calc/BAF_calc.py) is used for calculation copy numbers for genes of interest. It uses output from [FACETS](https://github.com/mskcc/facets) algorithm with SNP coverage). To run this script, you need **two files**:

1) pileup.txt: output from FACETS with SNP coverage;
2) .bed file: with genome coordinates for gene of interest

Arguments:

```
-c, --cov: path to FACETS pileup file;
-b, --bed-genes: path to .bed file with genes of interest coordinates;
-o, --output: path for output 
```  

Here is the example command for running this script:

```
python3 BAF_calc.py --cov data/sample1_pileup.txt 
		    --bed-genes interesting_genes.bed 
		    --output sample1_BAF.txt
```

The output will be generated in this format:

| gene | copy_num_mean | n |
|--------------|:-------:|:--------:|
| ERBB2 | - | 0 |
| BRCA1	| 5 | 8 |
| BRCA2	| 2 | 2 |
| CHEK1	| 1 | 1 |
| CHEK2	| - | 0 |
| EGFR	| 18 | 2 |
| ... | ... | ... |

## Future perspectives:

- Add GC correction
- Add correction for lower coverage on sides of targets
- Find ways for more precise copy number estimation
- Docker wrapper
