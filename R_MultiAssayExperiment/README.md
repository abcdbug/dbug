# MultiAssayExperiment

The `MultiAssayExperiment` class has been developed by the [Waldron Lab at CUNY](http://waldronlab.org/).

## Slides & more background info

The package, introduction and **slides** related to the tutorial on Nov 16, 2017 can all be found in the [Waldron Lab repo](https://github.com/waldronlab/MultiAssayExperimentWorkshop).

Here's the link to the [Bioconductor page](https://bioconductor.org/packages/release/bioc/html/MultiAssayExperiment.html) of the package.

And here is the [publication](http://cancerres.aacrjournals.org/content/77/21/e39) demonstrating the application of the package for data-driven cancer research.

![Cheatsheet](https://raw.githubusercontent.com/waldronlab/MultiAssayExperimentWorkshop/master/vignettes/MultiAssayExperiment_cheatsheet.pdf)

----------------------------------------

## Here's the gist of the `d:bug` session

* [Installation](#install)
* [Understanding the parts of the MAE](#MAE_subsets)
    - [`colData`](#clindat)
    - [observations/experiments](#assay)
    - [SampleMap](#sampleMap)
* [Constructing an MAE from scratch](#MAE)
    - [Subsetting the MAE](#subsetting)
* [Levi's R history from Nov 16](#LeviHistory)

In short, the `MultiAssayExperiment` class allows you to store different _types_ of data (e.g., RNA-seq, exome-seq, proteomics) for a set of specimens (e.g., a certain group of patients).

Every `MultiAssayExperiment` object consists of 3 types of data:

1. *experiments* 
	- used for storing the actual values/observations
	- think of it as a list of tables, one for every assay/type of experiment
	- can be accessed in numerous ways (see below), e.g. via `assays(mae)`, which will return a list of value matrices for every single experiment
	- the values for every type of assay can be supplied as simple matrices or `SummarizedExperiment` or any kind of data set that meets some basic requirements (see the slides for more info)
2. *colData* 
  - `DataFrame` (bioconductor's improved `data.frame`) with __information about the samples__, e.g., details about the patients, different mice, different time points - whatever you choose to be your smallest unit of interest
3. *sampleMap* 
  * this is what binds everything (i.e., the values from the assays and the sample information) together
  * for every single column of _every_ type of experiment, you this contains the information about which patient it belongs to, i.e., it provides the mapping between the row.names of `colData` (e.g. patient IDs) and the col.names of all the assays stored in `experiments`
  * needs three columns:
    * "assay": e.g., "RNAseq", "proteomics"
    * "primary": e.g. identifier of the patient or the mouse or the time point or whatever you choose as the content of the rows for `colData`)
    * "colname": the actual column names of every single assay of `experiments`

Here's my uglified version of [Marcel's elegant figure](http://cancerres.aacrjournals.org/content/canres/77/21/e39/F1.large.jpg) that tries to explains the relationships between the different data sets.

![](https://raw.githubusercontent.com/abcdbug/dbug/master/R_MultiAssayExperiment/mae_input.png)

The actual data that I used for the snapshots above can be found in the [xls sheet here](https://github.com/abcdbug/dbug/blob/master/R_MultiAssayExperiment/mae_example_data.xlsx).

The beauty of storing all of these data in one object is that you can use many functions and subsetting schemes across _all_ the data.
The general strategy is:

```
mae[ rows_of_experiments (e.g. genes), columns_of_experiments(e.g. patients), type_of_experiment ]
```

See below for more details.

<a id="install"></a>
### Installation and loading of the library and sample data

```
# install the package from bioconductor using biocLite
> library(MultiAssayExperiment)

# load a sample MultiAssayExperiment data set that comes with the package
> data("miniACC")

# have a look at the object
> miniACC
A MultiAssayExperiment object of 5 listed
 experiments with user-defined names and respective classes. 
 Containing an ExperimentList class object of length 5: 
 [1] RNASeq2GeneNorm: SummarizedExperiment with 198 rows and 79 columns 
 [2] gistict: SummarizedExperiment with 198 rows and 90 columns 
 [3] RPPAArray: SummarizedExperiment with 33 rows and 46 columns 
 [4] Mutations: matrix with 97 rows and 90 columns 
 [5] miRNASeqGene: SummarizedExperiment with 471 rows and 80 columns 
Features: 
 experiments() - obtain the ExperimentList instance 
 colData() - the primary/phenotype DataFrame 
 sampleMap() - the sample availability DataFrame 
 `$`, `[`, `[[` - extract colData columns, subset, or experiment 
 *Format() - convert into a long or wide DataFrame 
 assays() - convert ExperimentList to a SimpleList of matrices
```

There is data from 5 types of assay:

  - RNASeq2GeneNorm: gene mRNA abundance by RNA-seq
  - gistict: GISTIC genomic copy number by gene
  - RPPAArray: protein abundance by Reverse Phase Protein Array
  - Mutations: non-silent somatic mutations by gene
  - miRNASeqGene: microRNA abundance by microRNA-seq.

<a id="MAE_subsets"></a>
### The individual parts of the `MultiAssayExperiment` class

To get a feeling for what the different aspects look like, we took the object apart.

<a id="clindat"></a>
#### Get the patient info

```
clindat <- colData(miniACC)
```

Check the [clindat_colData.txt file](https://raw.githubusercontent.com/abcdbug/dbug/master/R_MultiAssayExperiment/clindat_colData.txt) above to see what that actually looks like.

<a id="assay"></a>
#### Extract the actual values (observations) from the different assay types 

Note the different ways to extract the actual data!

```
rnaseq <- assays(miniACC)[[1]] # assays returns a list of all assay data; you have to know that RNAseq is stored in the first one or explicitly specify the name
gistic <- miniACC[[2]] # again, you just know that the values for gistic are stored in the second array
mutations <- assay(miniACC, 4) # here we know that mutation values are stored in the fourth assay
protein <- miniACC[["RPPAArray"]] 
miRNAseq <- assays(miniACC)[["miRNASeqGene"]]
```

You can see some of the resulting files in this repo: [miRNAseq.txt](https://raw.githubusercontent.com/abcdbug/dbug/master/R_MultiAssayExperiment/miRnaseq.txt), [mutations.txt](https://raw.githubusercontent.com/abcdbug/dbug/master/R_MultiAssayExperiment/mutations.txt), [rnaseq.txt](https://raw.githubusercontent.com/abcdbug/dbug/master/R_MultiAssayExperiment/rnaseq.txt)

<a id="sampleMap"></a>
#### Extract the `sampleMap` 

```
map <- sampleMap(miniACC)
```

[sampleMap](https://raw.githubusercontent.com/abcdbug/dbug/master/R_MultiAssayExperiment/sampleMap.txt)

<a id="MAE"></a>
### Generate a `MultiAssayExperiment` object 

Now, let's assume you actually started the normal way, i.e., you have a bunch of text files and possibly spread sheets and you want to use those to generate a `MultiAssayExperiment`. You could easily re-create that using the files in this repo.

```
# colData
> my_clindat <- read.table("dbug/R_MultiAssayExperiment/clindat_colData.txt", 
                          sep = "\t", header = TRUE, stringsAsFactors = FALSE)
> rownames(my_clindat) <- my_clindat$patientID

# sampleMap
> my_map <- read.table("dbug/R_MultiAssayExperiment/sampleMap.txt", 
                      sep = "\t", header = TRUE, stringsAsFactors = FALSE)
> my_map$colname <- make.names(my_map$colname) # to make sure the hyphens are replaced with dots as is the case for the headers of my_clindat
> my_map_reduced <- subset(my_map, assay %in% c("RNASeq2GeneNorm", "Mutations"))  # I only want to use two assays for the example

# read in assay data for experiments
# note the stringsAsFactors=FALSE and the assignment of the row.names!
> my_rna <- read.table("dbug/R_MultiAssayExperiment/rnaseq.txt", 
                       sep = "\t", header = TRUE, row.names = 1, stringsAsFactors = FALSE)
> my_mutations <- read.table("dbug/R_MultiAssayExperiment/mutations.txt",
                       sep = "\t", header = TRUE, row.names = 1, stringsAsFactors = FALSE)

# let's make a list of all my assay data
> my_experiments <- list("RNASeq2GeneNorm" = my_rna,
                       "Mutations" = my_mutations)

# generate the MAE object!
> my_mae <- MultiAssayExperiment(experiments = my_experiments, 
                               sampleMap = my_map_reduced,
                               colData = my_clindat)
harmonizing input:
  removing 1 sampleMap rows with 'colname' not in colnames of experiments
> which(!my_map_reduced$colname %in% c(colnames(my_experiments$RNASeq2GeneNorm), colnames(my_experiments$Mutations)))
[1] 80
```
#### Subsetting fun <a id="subsetting"></a>

```
# let's retrieve the values from the RNAseq for the DIABLO gene for the first two patients
> assay(miniACC[ "DIABLO", 1:2 , "RNASeq2GeneNorm"])
harmonizing input:
  removing 13 colData rownames not in sampleMap 'primary'
harmonizing input:
  removing 77 sampleMap rows with 'colname' not in colnames of experiments
  removing 77 colData rownames not in sampleMap 'primary'
       TCGA-OR-A5J1-01A-11R-A29S-07 TCGA-OR-A5J2-01A-11R-A29S-07
DIABLO                     1719.294                     1141.489

# we can also retrieve ALL values for DIABLO for the first two patients (note that 
# we do not specify an assay of interest and use assays() instead of assay()
> assays(miniACC[ "DIABLO",1:2 , ])
harmonizing input:
  removing 376 sampleMap rows with 'colname' not in colnames of experiments
  removing 90 colData rownames not in sampleMap 'primary'
List of length 2
names(2): RNASeq2GeneNorm gistict
```

For more details, see the vignettes at the bioconductor page!

#### Levi's R history from the workshop <a id="LeviHistory"></a>

```
library(MultiAssayExperiment)
data(miniACC)
rnaseq <- assays(miniACC)[[1]]
gistic <- miniACC[[2]]
protein <- miniACC[["RPPAArray"]]
mutations <- assay(miniACC, 4)
clindat <- colData(miniACC)
map <- sampleMap(miniACC)
 
mylist <- list(RNASeq2GeneNorm=rnaseq,
               gistict=gistic,
               RPPAArray=protein,
               Mutations=mutations)
mae <- MultiAssayExperiment(experiments=mylist, colData=clindat, sampleMap=map)
mae["DIABLO", , ]
assays(mae["DIABLO",  , ])
assay(mae["DIABLO",  , ])
```

When asked about his "favorite function":

```
wideFormat(mae["DIABLO",  , ])
wideFormat(mae["DIABLO",  , ], colDataCols = "vital_status")
longFormat(mae["DIABLO",  , ], colDataCols = "vital_status")
```

