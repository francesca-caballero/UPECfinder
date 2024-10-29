# UPECfinder
An easy-to-use tool for in silico identification of uropathogenic Escherichia coli (UPEC) isolates.

## A Quick Note

The identification of virulence genes in UPEC is available as a web-service from CGE. The web-service works great, but is not the most efficient for 100s of genomes. This tool unifies the processes of identifying virulence genes and determining whether it is a UPEC.

## Introduction

`UPECfinder` is built using [camlhmp] (https://github.com/rpetit3/camlhmp) to identify Uropathogenic E. coli isolates. Using an input assembly (uncompressed or gzip-compressed), the sequences are blasted against a set of virulence genes. The genetic definition of UPEC consists of the presence of three or more of the genes vat, yfcV, chuA, and fyuA, according to Spurbeck et al.

## Installation

`UPECfinder` is available from Bioconda

### Conda

```{bash}
conda create -n UPECfinder -c conda-forge -c bioconda UPECfinder
conda activate UPECfinder
UPECfinder --help
```

## Usage

```{bash}
Usage: camlhmp-blast-regions [OPTIONS]

🐪 camlhmp-blast-regions 🐪 - Classify assemblies with a camlhmp schema using BLAST against larger genomic regions

╭─ Options ─────────────────────────────────────────────────────────────────────────────────────╮
│ *  --input         -i  TEXT     Input file in FASTA format to classify [required]             │
│ *  --yaml          -y  TEXT     YAML file documenting the targets and types                   │
│                                 [default: bin/../data/upec.yaml]                              │
│                                 [required]                                                    │
│ *  --targets       -t  TEXT     Query targets in FASTA format                                 │
│                                 [default: bin/../data/targets.fasta]                          │
│                                 [required]                                                    │
│    --outdir        -o  PATH     Directory to write output [default: ./]                       │
│    --prefix        -p  TEXT     Prefix to use for output files [default: camlhmp]             │
│    --min-pident        INTEGER  Minimum percent identity to count a hit [default: 90]         │
│    --min-coverage      INTEGER  Minimum percent coverage to count a hit [default: 90]         │
│    --force                      Overwrite existing reports                                    │
│    --verbose                    Increase the verbosity of output                              │
│    --silent                     Only critical errors will be printed                          │
│    --version       -V           Print schema and camlhmp version                              │
│    --help                       Show this message and exit.                                   │
╰───────────────────────────────────────────────────────────────────────────────────────────────╯
```

### --input

An assembly in FASTA format (compressed with gzip, or uncompressed) to identify whether it is a UPEC.

### --yaml & --targets

__*Note: These should be automatically set for you!*__

These are the required inputs for camlhmp to specify the schema and targets for identifying UPEC.

### --prefix

The prefix to use for the output files. If a prefix is not given, the basename of the input assembly will be used.

### --outdir

The directory to save result files. If the directory does not exist it will be created.

### --min-pident

__*Note: These should be automatically set for you!*__

The minimum percent identity for a blast hit to be considered. This is compared against the `pident` column of the blast output.

### --min-coverage

__*Note: These should be automatically set for you!*__

The minimum coverage of a gene to be considered. This calculation does include mismatches and gaps, but those should be expected to be minimal if --min_pident is set to the default (95%) or similar.

## Output Files

| Filename               | Description                                          |
|------------------------|------------------------------------------------------|
| `{PREFIX}.tsv`         | A tab-delimited file with the predicted serogroup    |
| `{PREFIX}.blastn.tsv`  | A tab-delimited file of all blast hits               |
| `{PREFIX}.details.tsv` | A tab-delimited file with details for each serogroup |

### Example `{PREFIX}.tsv`

This file will contain the final predicted type based on highest coverage. Here's what to expect the output to look like:

```{bash}
sample	type	targets	coverage	hits	schema	schema_version	camlhmp_version	params	comment
1366	UPEC vat/yfcV/chuA/fyuA	vat:2:AY151282,yfcV:37:UGFD01000001,chuA:10:UFZA01000001,fyuA:96:UGGK01000002	100.00,100.00,100.00,100.00	1,1,1,1	upec_finder	0.0.1	1.1.0	min-coverage=90;min-pident=90
```

| Column Name     | Description                                                              |
|-----------------|--------------------------------------------------------------------------|
| sample          | Name of the sample processed                                             |
| type            | The predicted type                                                       |
| targets         | The targets for the type with a passing hit                              |
| coverage        | The percent of the sequences that was aligned to                         |
| hits            | The number of blast hits included in the prediction (_fewer the better_) |
| schema          | The schema used for the prediction                                       |
| schema_version  | The version of the schema used for the prediction                        |
| camlhmp_version | The version of camlhmp used for the prediction                           |
| params          | The parameters used for the prediction                                   |
| comment         | Any comments produced during analysis                                    |

### Example `{PREFIX}.blastn.tsv`

The blast results are in standard `-outfmt 6`, and will be similar to this:

```{bash}
qseqid	sseqid	pident	qcovs	qlen	slen	length	nident	mismatch	gapopen	qstart	qend	sstart	send	evalue	bitscore
vat:2:AY151282	1366_NODE_3_length_594660_cov_59.710600	99.589	100	4134	594660	4137	4120	9	7	1	4134	94432	90301	0.0	7539
yfcV:37:UGFD01000001	1366_NODE_5_length_359795_cov_56.776010	96.641	100	750	359795	774	748	2	5	1	750	29223	29996	0.0	1264
chuA:10:UFZA01000001	1366_NODE_9_length_203436_cov_73.223925	98.336	100	1983	203436	1983	1950	33	0	1	1983	17378	19360	0.0	3480
fyuA:96:UGGK01000002	1366_NODE_1_length_636446_cov_48.946104	99.903	100	2055	636446	2056	2054	1	1	1	2055	102614	100559	0.0	3784
```

### Example `{PREFIX}.details.tsv`

This file provides the the coverage and number of hits for each of the types. This can be useful for a deeper review, and it should look like the following:

```{bash}
sample	type	status	targets	missing	coverage	hits	schema	schema_version	camlhmp_version	params	comment
1366	UPEC vat/yfcV/chuA	FALSE	vat:2:AY151282,yfcV:37:UGFD01000001,chuA:10:UFZA01000001		100.00,100.00,100.00	1,1,1	upec_finder	0.0.1	1.1.0	min-coverage=90;min-pident=90	Excluded target fyuA:96:UGGK01000002 found, failing type UPEC vat/yfcV/chuA
1366	UPEC vat/yfcV/fyuA	FALSE	vat:2:AY151282,yfcV:37:UGFD01000001,fyuA:96:UGGK01000002		100.00,100.00,100.00	1,1,1	upec_finder	0.0.1	1.1.0	min-coverage=90;min-pident=90	Excluded target chuA:10:UFZA01000001 found, failing type UPEC vat/yfcV/fyuA
1366	UPEC yfcV/fyuA/chuA	FALSE	yfcV:37:UGFD01000001,fyuA:96:UGGK01000002,chuA:10:UFZA01000001		100.00,100.00,100.00	1,1,1	upec_finder	0.0.1	1.1.0	min-coverage=90;min-pident=90	Excluded target vat:2:AY151282 found, failing type UPEC yfcV/fyuA/chuA
1366	UPEC vat/chuA/fyuA	FALSE	vat:2:AY151282,chuA:10:UFZA01000001,fyuA:96:UGGK01000002		100.00,100.00,100.00	1,1,1	upec_finder	0.0.1	1.1.0	min-coverage=90;min-pident=90	Excluded target yfcV:37:UGFD01000001 found, failing type UPEC vat/chuA/fyuA
1366	UPEC vat/yfcV/chuA/fyuA	TRUE	vat:2:AY151282,yfcV:37:UGFD01000001,chuA:10:UFZA01000001,fyuA:96:UGGK01000002		100.00,100.00,100.00,100.00	1,1,1,1	upec_finder	0.0.1	1.1.0	min-coverage=90;min-pident=90	<img width="1618" alt="image" src="https://github.com/user-attachments/assets/eb662437-15d3-44d0-87ae-aa7f6f7b04b7">
```

| Column Name     | Description                                                              |
|-----------------|--------------------------------------------------------------------------|
| sample          | Name of the sample processed                                             |
| type            | The predicted type                                                       |
| status          | True or false for each type                                              |
| targets         | The targets for the type with a passing hit                              |
| missing         | Genes not found in the BLAST search                                      | 
| coverage        | The percent of the sequences that was aligned to                         |
| hits            | The number of blast hits included in the prediction (_fewer the better_) |
| schema          | The schema used for the prediction                                       |
| schema_version  | The version of the schema used for the prediction                        |
| camlhmp_version | The version of camlhmp used for the prediction                           |
| params          | The parameters used for the prediction                                   |
| comment         | Any comments produced during analysis                                    |

## Naming

A self-explanatory name: if you are trying to find UPEC, use `UPECfinder`.

### License Choice

I selected Apache 2.0 as the license to match other CGE tools.

## Citation

If you use this tool please cite the following:

__camlhmp__
Petit III RA camlhmp: Classification through yAML Heuristic Mapping Protocol (GitHub)
 
__BLAST+__
Camacho C, Coulouris G, Avagyan V, Ma N, Papadopoulos J, Bealer K, Madden TL BLAST+: architecture and applications. BMC Bioinformatics 10, 421 (2009)
 
__UPEC identification method__
Spurbeck RR, Dinh PC Jr, Walk ST, Stapleton AE, Hooton TM, Nolan LK, Kim KS, Johnson JR, Mobley HL. Escherichia coli isolates that carry vat, fyuA, chuA, and yfcV efficiently colonize the urinary tract. Infect Immun. 2012 Dec;80(12):4115-22. doi: 10.1128/IAI.00752-12. Epub 2012 Sep 10. PMID: 22966046; PMCID: PMC3497434.
