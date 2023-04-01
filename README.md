# *CENTRE - Short description*
CENTRE (Cell-type-specific ENhancer-Target pREdiction) is a tool for identifying and characterizing gene enhancer interactions.
By combining generic features and cell type specific features, CENTRE is able to
classify and predict active gene enhancer pairs in the given cell type with high accuracy
and with fewer input data than state of the art methods.

### Contact

lopez_s@molgen.mpg.de

### Citation


## Requirements
- R (tested 4.0.0)
- crupR
- GenomicRanges and IRanges
- metapod
- RSQLite
- xgboost

## User Provided Data

CENTRE computes features based on Histone ChIP-seq (H3K27ac, H3K4me3 and H3K4me1
) data and RNA-seq data that the user provides. As well as, a dataframe with the 
genes of interest or the genes and enhancer pairs of interest.

User data : 

- Histone ChIP-seq in BAM format for H3K27ac, H3K4me3 and H3K4me1. Additionally, 
a Control ChIP-seq experiment to go with the HM ChIP-seq is strongly advised but
CENTRE can also run without it.
- RNA-seq ENCODE4 gene quantifications in an R dataframe. This dataframe will have three
columns one 
 with the ENSEMBL ID's, transcript ID's and one with the TPM values. 
- The ID's to either the enhancers and genes or just the genes in an R dataframe.

## CENTRE Provided Data

CENTRE uses precomputed datasets that the user needs to download either by using
the `CENTRE::downloadPredcomputedData()` or downloading the data from http://owww.molgen.mpg.de/~CENTRE_data/PrecomputedData.db
and adding it to the /inst/extdata folder. 

The downloaded PrecomputedData.db is a database containing the data to the 
precomputed Wilcoxon rank sum tests on the following data sets:
- CAGE-seq dataset 
- DNAse hypersensitivity dataset
- DNAse-seq gene expression dataset
- CRUP-EP gene expression dataset
As well as: 
- Pearson Correlations between CRUP-EP(Enhancer Probability) and CRUP-PP
(Promoter Probability) across 104 cell types

For more information on what these datasets are we recommend checking the 
publication.

## How to run CENTRE
### Installing CENTRE
This package is still in development. To install it correctly do the
following: 
1. Clone the repository
2. Download the sysdata.rda from http://owww.molgen.mpg.de/~CENTRE_data/hello.html
and move it into the CENTRE/R folder
3. Install the R package *after* downloading the sysdata.rda either from
terminal or using `devtools::build()` and ``devtools::install()`
4. After installing the R package use our `CENTRE::downloadPrecomputedData()` 
function to download the PrecomputedData.db 

You can find the data for the example below at https://nc.molgen.mpg.de/cloud/index.php/s/TP4ogT5b3Xewnzy

Note: sysdata.rda contains the annotations for GENCODE v40 and ENCODE cCREs v3
we are working to make them available through AnnotationHub. Sorry for the inconvinience 
of having to install it manually.

### General workflow

Two use cases : 

- User has only genes: `createPairs()` -> `computeGenericFeatures()` -> 
`computeCellTypeFeatures()` -> `centreClassification()`

- User has gene enhancer pairs : `computeGenericFeatures()` -> 
`computeCellTypeFeatures()` -> `centreClassification()`

### Create Pairs
The function `createPairs(genes)` finds all enhancers in 500KB from the 
transcription start site of the genes provided and creates all possible enhancer 
gene pairs.

It takes as input a dataframe `genes` which has one column with the ENSEMBL 
gene ID's. 

Example of the format:

|gene_id|
|-------|
|ENSG00000269831.1|
|ENSG00000131584.14|
|ENSG00000235146.2|
| ... |

Returns a dataframe of two columns with the gene ID's (without version 
identifier) and the corresponding 
ENCODE cCREs enhancer id's

|gene_id1 | enhancer_id|
|---------|------------|
|ENSG00000269831 | EH38E1310357|
|ENSG00000131584 | EH38E1310357|
|ENSG00000235146 | EH38E1310357|
| ... | ... |


### Compute generic features
The function `computeGenericFeatures(pairs)` computes the features that are not 
cell type specific. 

Takes as input a dataframe `pairs` with a format like the one in the last table.
The function returns a dataframe with the following features as columns: 

- `distance`: the distance between the middle point of the enhancer and the 
transcription start site of the gene in each pair
- `cor_CRUP`: the CRUP correlation for each pair. This is one of the precomputed
datasets.
- `combined_tests`: the combined p-values of all of the precomputed Wilcoxon 
tests.

### Compute cell type specific features
The function 
`computeCellTypeFeatures(metaData, condition , replicate, mapq, input.free, sequencing, tpm, featuresGeneric)` 
computes the features that are cell type specific. 

Takes as input the following:

- `metaData`: Dataframe with the path to the cell type specific ChIP-seq
experiments.
Information on the format of this dataframe can be found in the manuals of crupR.
- `condition`: The number of conditions that need to be normalized by crupR.
- `replicate`: The number of replicates that need to be normalized by crupR.
- `mapq`: Integer Minimum mapping quality of the reads that should be included
in the crupR normalization. Default is 10.
- `input.free`: Boolean value indicating whether a Control/Input ChIP-seq
experiment is provided to go with the Histone Modification ChIP-seq experiments.
If the parameter is set to FALSE crupR normalization will be run in input.free
mode.
- `cores`: Integer indicating how many cores CRUP should be run with.
- `sequencing`: String "single" or "paired" indicating the type of sequencing of
the histone ChIP-seq experiments.
-  `tpm`: RNA-seq ENCODE4 gene quantification data in R dataframe format. The R 
dataframe has to have 3 columns, one for the `gene_id`, 
one for the `transcript_ids` and one for the TPM value.
- `features_generic`: The dataframe that was produced in `computeGenericFeatures()`

Example of the tpm dataframe: 

|gene_id | transcript_id.s.| TPM|
|--------|-----------------|----|
|   10904 |           10904|   0|
|   12954|            12954|   0|
|   12956|            12956|   0|
|   ... |            ...|   0|




Returns the cell type specific features : 

- `EP_prob_enh`: CRUP-EP(Enhancer Probability) for Enhancers
- `EP_prob_gene`: CRUP-EP(Enhancer Probability) for Promoters
- `reg_dist_enh`: Regulatory distance computed on enhancer probabilities
- `norm_reg_dist_enh`: Normalized regulatory distance computed on enhancer 
probabilities
- `PP_prob_enh`: CRUP-PP(Promoter Probability) for Enhancers
- `PP_prob_gene`: CRUP-PP(Promoter Probability) for Promoters
- `reg_dist_enh`: Regulatory distance computed on promoter probabilities
- `norm_reg_dist_enh`: Normalized regulatory distance computed on promoter probabilities
- `RNA_seq`: RNA-seq TPM values for the gene in each pair of the cell type of interest.

Again, for the meaning of regulatory distance and the other features we recommend
checking the citation.

### Classify Gene Enhancer Pairs
With the function `centrePrediction(features_generic, features_celltype, model)`
the gene enhancer targets are classified as active or inactive. 
The function takes as input the generic features, the cell type specific features
and the model (CENTRE model by default).

Returns a dataframe with the `pairs` ID (`enhancer_id` and `gene_id` concatenated
by a `_` of the corresponding pair and the label and probability for each of the pairs.

### Example run

We provide an in-package example on the cell line HeLa-S3. The Histone Mark data 
and RNA-seq data are from ENCODE :
| Experiment type| ENCODE experiment accession | File accession numbers|
|----------------|-----------------------------|-----------------------|
| H3K4me1        |ENCSR000APW| ENCFF712AAP, ENCFF826OLG|
| H3K4me3        | ENCSR340WQU|ENCFF650IXI, ENCFF760VTC |
| H3K27ac        |ENCSR000AOC |	ENCFF609ZAE, ENCFF711QAI |
| ChIP-seq Control| ENCSR000AOB|ENCFF017QCL, ENCFF842IEZ|
| RNA-seq        | ENCSR000CPP |ENCFF297BJF, ENCFF623UDU|

For the Histone ChIP-seq replicates were merged using bamtools merge and for the
RNA-seq data we take the mean of the TPM values over the replicates.

```
#Install the package
devtools::install_github("slrvv/CENTRE")

#Download PrecomputedData (only once after installation)

downloadPrecomputedData(method = "wget")
#Make sure that the method of download you choose (wget, libcurl, curl, wininet
#etc) is available in your computer.
#If the file is downloaded succesfully it will return 0

#Computation:
#Start by providing genes with their ENSEMBL id
candidates <- readRDS(file= system.file("extdata",
                                         "input_generic_features.rds",
                                         package = "CENTRE"))
genes <- as.data.frame(candidates[, 1])



#Remember to give the columns the name "gene_id" 
colnames(genes) <- c("gene_id")

#Generate the candidate pairs
candidate_pairs <- createPairs(genes)

#Compute the generic features for given names

generic_features <- computeGenericFeatures(candidate_pairs)

## Prepare the data needed for computing cell type features

files <- c(system.file("extdata/example","HeLa_H3K4me1.bam", package = "CENTRE"),
          system.file("extdata/example","HeLa_H3K4me3.bam", package = "CENTRE"),
          system.file("extdata/example","HeLa_H3K27ac.bam", package = "CENTRE")
          
# Control ChIP-seq experiment to go with the rest of ChIP-seqs
inputs <- system.file("extdata/example", "HeLa_input.bam", package = "CENTRE")

metaData <- data.frame(HM = c("H3K4me1", "H3K4me3", "H3K27ac"),
                       condition = c(1, 1, 1), replicate = c(1, 1, 1),
                       bamFile = files, inputFile = rep(inputs, 3))
#More information on this step is found in the crupR documentation

tpmfile <- read.table(system.file("extdata/example", "HeLa-S3.tsv", package = "CENTRE"),
                      sep = "", stringsAsFactors = F, header = T)

celltype_features <- computeCellTypeFeatures(metaData,
                                    condition = 1,
                                    replicate = 1,
                                    mapq = 10,
                                    input.free = FALSE,
                                    cores = 1,
                                    sequencing = "single",
                                    tpmfile = tpmfile,
                                    featuresGeneric = generic_features)
  
# Finally compute the predictions
predictions <- centrePrediction(celltype_features,
                                  generic_features)
  

```
