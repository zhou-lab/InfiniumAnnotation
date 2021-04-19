# Mammal40 Array

see also [other infinium arrays](../README.md)

## [Manifest (based on xxx assembly)]()

[Column Header Specification]().

#### [Mammal Array ID system]()

## [Gene Association]()

Gene association of probes based on (GENCODE vM25)? transcript definition. This file includes annotation of probes if they fall from 1.5kbp upstream Transcription Start Site (TSS/Promoter), to Transcription Termination Site. A probe can be considered promoter associated if it is located from 1.5kbps upstream TSS to 1.5kbps dwonstream TSS. This information is given in the distToTSS column. Otherwise, the probe is considered associated with gene body. All isoforms are considered for each gene.

## Epigenomic Feature Annotation

### [ChromHMM]()

### Histone Modification 

Histone modification data was obtained through the ENCODE database. Each tsv.gz file is identified by its GSM ID corresponding to the sample from the Gene Expression Omnibus (GEO).
 
#### Column Description
- 1: Mammal40 Illumina probe ID
- 2: Histone ChIP-seq signal peaks

### Transcription Factor Binding Site 

Transcription Factor Binding Site (TFBS) data was obtained through the ENCODE database. Each tsv.gz file is identified by its GSM ID corresponding to the sample from GEO.

#### Column Description
- 1: Mammal40 Illumina probe ID
- 2: TF ChIP-seq signal peaks

## Using SeSAMe for preprocessing Infinium Mouse Array

Please note that you need sesame (Bioconductor link: [stable release](https://bioconductor.org/packages/release/bioc/html/sesame.html), [development](https://bioconductor.org/packages/devel/bioc/html/sesame.html)) version 1.9+ (currently on development branch, which needs [R-4.1](https://cran.r-project.org/bin/windows/base/rdevel.html)) for native mouse array support. You can simply call the openSesame pipeline

```R
library(sesame)
betas = openSesame("IDAT_folder")
```

More information can be found at the sesame [mouse array vignette](https://bioconductor.org/packages/devel/bioc/vignettes/sesame/inst/doc/mammal.html).

For previous versions, you need to supply the custom sesame order file for mouse array. Here is an example of using custom order file.

```R
library(sesame)
mft <- readRDS(url(""))
ssets <- lapply(searchIDATprefixes('path_to_IDAT_folder'), readIDATpair, manifest=mft$ordering, controls=mft$controls, platform='Mammal40')
```

## Illumina manifest

#### [Support Document Download Page]()

#### [Manifest File]()

## Reference

_in submission_
