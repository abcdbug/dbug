# MultiAssayExperiment

The `MultiAssayExperiment` class has been developed by the [Waldron Lab at CUNY](http://waldronlab.org/).

## Slides & more background info

The package, introduction and **slides** related to the tutorial on Nov 16, 2017 can all be found in the [Waldron Lab repo](https://github.com/waldronlab/MultiAssayExperimentWorkshop).

Here's the link to the [Bioconductor page](https://bioconductor.org/packages/release/bioc/html/MultiAssayExperiment.html) of the package.

And here is the [publication](http://cancerres.aacrjournals.org/content/77/21/e39) demonstrating the application of the package for data-driven cancer research.

----------------------------------------

## Here's the gist of the `d:bug` session:

In short, the `MultiAssayExperiment` class allows you to store different _types_ of data (e.g., RNA-seq, exome-seq, proteomics) for a set of speciments (e.g., a certain group of patients).

Every `MultiAssayExperiment` object consists of 3 types of data:

1. *experiments* (--> will be subsettable as the "rows" of the final `MultiAssayExperiment` object)
	- used for storing the actual values/observations
	- think of it as a list of tables, one for every assay/type of experiment
	- can be accessed in numerous ways (see below), e.g. via `assays(mae)`, which will return a list of value matrices for every single experiment
	- the values for every type of assay can be supplied as simple matrices or `SummarizedExperiment` or any kind of data set that meets some basic requirements (see the slides for more info)
2. *colData* (--> will define what will be subsettable as the "columns" of the final `MultiAssayExperiment` object)
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

  -  RNASeq2GeneNorm: gene mRNA abundance by RNA-seq
  - gistict: GISTIC genomic copy number by gene
  - RPPAArray: protein abundance by Reverse Phase Protein Array
  - Mutations: non-silent somatic mutations by gene
  - miRNASeqGene: microRNA abundance by microRNA-seq.


### The individual parts of the `MultiAssayExperiment` class

To get a feeling for what the different aspects look like, we took the object apart.

#### get the patient info

```
> clindat <- colData(miniACC)
> head(clindat)
DataFrame with 6 rows and 30 columns
                patientID years_to_birth vital_status days_to_death days_to_last_followup tumor_tissue_site
              <character>      <integer>    <integer>     <integer>             <integer>       <character>
TCGA-OR-A5J1 TCGA-OR-A5J1             58            1          1355                    NA           adrenal
TCGA-OR-A5J2 TCGA-OR-A5J2             44            1          1677                    NA           adrenal
TCGA-OR-A5J3 TCGA-OR-A5J3             23            0            NA                  2091           adrenal
TCGA-OR-A5J4 TCGA-OR-A5J4             23            1           423                    NA           adrenal
TCGA-OR-A5J5 TCGA-OR-A5J5             30            1           365                    NA           adrenal
TCGA-OR-A5J6 TCGA-OR-A5J6             29            0            NA                  2703           adrenal
             pathologic_stage pathology_T_stage pathology_N_stage      gender date_of_initial_pathologic_diagnosis
                  <character>       <character>       <character> <character>                            <integer>
TCGA-OR-A5J1         stage ii                t2                n0        male                                 2000
TCGA-OR-A5J2         stage iv                t3                n0      female                                 2004
TCGA-OR-A5J3        stage iii                t3                n0      female                                 2008
TCGA-OR-A5J4         stage iv                t3                n1      female                                 2000
TCGA-OR-A5J5        stage iii                t4                n0        male                                 2000
TCGA-OR-A5J6         stage ii                t2                n0      female                                 2006
             radiation_therapy                    histological_type residual_tumor number_of_lymph_nodes
                   <character>                          <character>    <character>             <integer>
TCGA-OR-A5J1                no adrenocortical carcinoma- usual type             r0                    NA
TCGA-OR-A5J2                no adrenocortical carcinoma- usual type             r2                     0
TCGA-OR-A5J3                no adrenocortical carcinoma- usual type             r0                     0
TCGA-OR-A5J4                no adrenocortical carcinoma- usual type             r2                     2
TCGA-OR-A5J5                no adrenocortical carcinoma- usual type             r2                    NA
TCGA-OR-A5J6                no adrenocortical carcinoma- usual type             r0                    NA
                                  race          ethnicity   Histology     C1A.C1B                              mRNA_K4
                           <character>        <character> <character> <character>                          <character>
TCGA-OR-A5J1                     white                 NA  Usual Type         C1A steroid-phenotype-high+proliferation
TCGA-OR-A5J2                     white hispanic or latino  Usual Type         C1A steroid-phenotype-high+proliferation
TCGA-OR-A5J3                     white hispanic or latino  Usual Type         C1A               steroid-phenotype-high
TCGA-OR-A5J4                     white hispanic or latino  Usual Type          NA                                   NA
TCGA-OR-A5J5                     white hispanic or latino  Usual Type         C1A               steroid-phenotype-high
TCGA-OR-A5J6 black or african american hispanic or latino  Usual Type         C1B                steroid-phenotype-low
                    MethyLevel miRNA.cluster SCNA.cluster protein.cluster         COC    OncoSign    purity    ploidy
                   <character>   <character>  <character>       <integer> <character> <character> <numeric> <numeric>
TCGA-OR-A5J1         CIMP-high       miRNA_1        Quiet              NA        COC3         CN2      0.90      1.95
TCGA-OR-A5J2          CIMP-low       miRNA_1        Noisy               1        COC3    TP53/NF1      0.89      1.31
TCGA-OR-A5J3 CIMP-intermediate       miRNA_6  Chromosomal               3        COC2         CN2      0.93      1.25
TCGA-OR-A5J4         CIMP-high       miRNA_6  Chromosomal              NA          NA         CN1      0.87      2.60
TCGA-OR-A5J5 CIMP-intermediate       miRNA_2  Chromosomal              NA        COC2    TP53/NF1      0.93      2.75
TCGA-OR-A5J6          CIMP-low       miRNA_1        Noisy               2        COC1  TERT/ZNRF3      0.68      3.32
             genome_doublings       ADS
                    <integer> <numeric>
TCGA-OR-A5J1                0     -0.08
TCGA-OR-A5J2                0     -0.84
TCGA-OR-A5J3                0      1.18
TCGA-OR-A5J4                1        NA
TCGA-OR-A5J5                1     -1.00
TCGA-OR-A5J6                1      1.11
```

#### extract the actual values (observations) from the different assay types

Note the different ways to extract the actual data!

```
rnaseq <- assays(miniACC)[[1]] # assays returns a list of all assay data; you have to know that RNAseq is stored in the first one
gistic <- miniACC[[2]]
protein <- miniACC[["RPPAArray"]]
mutations <- assay(miniACC, 4)
```

```
map <- sampleMap(miniACC)
```

#### generate a `MultiAssayExperiment` object

```
unique(map$assay)
```

![Cheatsheet](https://raw.githubusercontent.com/waldronlab/MultiAssayExperimentWorkshop/master/vignettes/MultiAssayExperiment_cheatsheet.pdf)
