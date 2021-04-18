# MouseMethylation (MM285)

see also [other infinium arrays](README.md)

## [Manifest (based on mm10 assembly)](https://zwdzwd.s3.amazonaws.com/InfiniumAnnotation/current/MM285/MM285.mm10.manifest.tsv.gz)

[Column Header Specification](20210418_manifest_column_specs.md).

#### [Mouse Array ID system](20210418_manifest_column_specs.md)

The mouse array uses an improved ID system that is built on top of the conventional "cg" numbers that EPIC and HM450 human array uses. The new ID system uniquely specifies the design details. The new ID system is designed to accomodate more flexible probe design such as replicates and opposite-strand design. See [here](20210418_manifest_column_specs.md) for detail.

## [Manifest (based on mm39 assembly)](https://zwdzwd.s3.amazonaws.com/InfiniumAnnotation/current/MM285/MM285.mm39.manifest.tsv.gz)

## [Gene Association](https://zwdzwd.s3.amazonaws.com/InfiniumAnnotation/current/MM285/MM285.mm10.manifest.gencode.vM25.tsv.gz)

Gene association of probes based on GENCODE vM25 transcript definition. This file includes annotation of probes if they fall from 1.5kbp upstream Transcription Start Site (TSS/Promoter), to Transcription Termination Site. A probe can be considered promoter associated if it is located from 1.5kbps upstream TSS to 1.5kbps dwonstream TSS. This information is given in the distToTSS column. Otherwise, the probe is considered associated with gene body. All isoforms are considered for each gene.

## Epigenomic Feature Overlap

### ChromHMM (TODO)

### Histone Modification 

Histone modification data was obtained through the ENCODE database. Each tsv.gz file is identified by its GSM ID corresponding to the sample from the Gene Expression Omnibus (GEO).
 
#### Column Description
- 1: MM285 Illumina probe ID
- 2: Histone ChIP-seq signal peaks

### Transcription Factor Binding Site 

Transcription Factor Binding Site (TFBS) data was obtained through the ENCODE database. Each tsv.gz file is identified by its GSM ID corresponding to the sample from GEO.

#### Column Description
- 1: MM285 Illumina probe ID
- 2: TF ChIP-seq signal peaks

## Using SeSAMe for preprocessing Infinium Mouse Array

Please note that you need sesame (Bioconductor link: [stable release](https://bioconductor.org/packages/release/bioc/html/sesame.html), [development](https://bioconductor.org/packages/devel/bioc/html/sesame.html)) version 1.9+ (currently on development branch, which needs [R-4.1](https://cran.r-project.org/bin/windows/base/rdevel.html)) for native mouse array support. You can simply call the openSesame pipeline

```R
library(sesame)
betas = openSesame("IDAT_folder")
```

More information can be found at the sesame [mouse array vignette](https://bioconductor.org/packages/devel/bioc/vignettes/sesame/inst/doc/mouse.html).

For previous versions, you need to supply the custom sesame order file for mouse array. Here is an example of using custom order file.

```R
library(sesame)
mft <- readRDS(url("https://zwdzwd.s3.amazonaws.com/InfiniumAnnotation/current/MM285/MM285.address.rds"))
ssets <- lapply(searchIDATprefixes('path_to_IDAT_folder'), readIDATpair, manifest=mft$ordering, controls=mft$controls, platform='MM285')
```

## Illumina manifest

#### [Support Document Download Page](https://support.illumina.com/downloads/infinium-mouse-methylation-manifest-file-csv.html)

#### [Manifest File](https://support.illumina.com/content/dam/illumina-support/documents/downloads/productfiles/mouse-methylation/Infinium%20Mouse%20Methylation%20v1.0%20A1%20GS%20Manifest%20File.csv)

## Reference

_in submission_