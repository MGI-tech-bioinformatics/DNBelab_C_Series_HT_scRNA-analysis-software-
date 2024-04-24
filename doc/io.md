## How to read output results for analysis in R or Python.



### scRNA



#### seurat

```R
library(Seurat)
counts.data <- Read10X(data.dir = "/output/filter_matrix",gene.column = 1)
```

Using functions to read.(Applicable to raw_matrix, filter_matrix, and RNAvelocity_matrix, etc.)

```R
ReadMatrix_C4 <- function(mex_dir=NULL,
                     barcode.path = NULL,
                     feature.path = NULL,
                     matrix.path=NULL,
                     use_10X=FALSE) {
  if (is.null(mex_dir) && is.null(barcode.path)  && is.null(feature.path) &&
        is.null(matrix.path)) {
    stop("No matrix set.")
  }
  if (!is.null(mex_dir) && !file.exists(mex_dir) ) {
    stop(paste0(mex_dir, " does not exist."))
  }
  if (is.null(barcode.path)  && is.null(feature.path) && is.null(matrix.path)) {
    barcode.path <- paste0(mex_dir, "/barcodes.tsv.gz")
    feature.path <- paste0(mex_dir, "/features.tsv.gz")
    matrix.path <- paste0(mex_dir, "/matrix.mtx.gz")
  }
  spliced.path <- paste0(mex_dir, "/spliced.mtx.gz")
  unspliced.path <- paste0(mex_dir, "/unspliced.mtx.gz")
  spanning.path <- paste0(mex_dir, "/spanning.mtx.gz")

  if (!file.exists(barcode.path) || !file.exists(feature.path)) {
    stop(paste0("No expression file found at ", mex_dir))
  }

  .ReadPISA0 <- function(barcode.path, feature.path, matrix.path, use_10X) {
    mat <- Matrix::readMM(file = matrix.path)
    feature.names <- read.delim(feature.path,
                                header = FALSE,
                                stringsAsFactors = FALSE
                                )
    barcode.names <- read.delim(barcode.path,
                                header = FALSE,
                                stringsAsFactors = FALSE
                                )
    colnames(mat) <- barcode.names$V1
    if (use_10X == TRUE) {
      rownames(mat) <- make.unique(feature.names$V2)
    } else {
      rownames(mat) <- make.unique(feature.names$V1)
    }
    mat
  }
  
  if (!file.exists(spliced.path) && file.exists(matrix.path)) {
    return(.ReadPISA0(barcode.path, feature.path, matrix.path, use_10X))
  }
  mat <- list()
  cat("Load spliced matrix ...\n")
  mat$spliced <- Matrix::readMM(file = spliced.path)
  cat("Load unspliced matrix ...\n")
  mat$unspliced <- Matrix::readMM(file = unspliced.path)
  cat("Load spanning matrix ...\n")
  mat$spanning <- Matrix::readMM(file = spanning.path)

  feature.names <- read.delim(feature.path,
                              header = FALSE,
                              stringsAsFactors = FALSE
  )
  barcode.names <- read.delim(barcode.path,
                              header = FALSE,
                              stringsAsFactors = FALSE
  )
  colnames(mat$spliced) <- barcode.names$V1
  rownames(mat$spliced) <- make.unique(feature.names$V1)
  colnames(mat$unspliced) <- barcode.names$V1
  rownames(mat$unspliced) <- make.unique(feature.names$V1)
  colnames(mat$spanning) <- barcode.names$V1
  rownames(mat$spanning) <- make.unique(feature.names$V1)
  mat
}
```



#### scanpy

```python
import scanpy as sc
adata = sc.read_h5ad('/output/filter_feature.h5ad')
```

Using functions to read.

```python
import pandas as pd
import scipy.io
import anndata
from scipy.sparse import csr_matrix

def read_anndata_C4(path):   
    mat = scipy.io.mmread(os.path.join(path, "matrix.mtx.gz")).astype("float32")
    mat = mat.transpose()
    mat = csr_matrix(mat)
    adata = anndata.AnnData(mat,dtype="float32")
    genes = pd.read_csv(os.path.join(path, "features.tsv.gz"), header=None, sep='\t')
    var_names = genes[0].values
    var_names = anndata.utils.make_index_unique(pd.Index(var_names))
    adata.var_names = var_names
    adata.var['gene_symbols'] = genes[0].values
    adata.obs_names = pd.read_csv(os.path.join(path, "barcodes.tsv.gz"), header=None)[0].values
    adata.var_names_make_unique()
    return adata
```


</br>
</br>

### scATAC

#### signac

```R
require(magrittr)
require(readr)
require(Matrix)
require(tidyr)
require(dplyr)
 
mex_dir_path <- "/output/filter_peak_matrix"
mtx_path <- paste(mex_dir_path, "matrix.mtx.gz", sep = '/')
feature_path <- paste(mex_dir_path, "peaks.bed.gz", sep = '/')
barcode_path <- paste(mex_dir_path, "barcodes.tsv.gz", sep = '/')
 
features <- readr::read_tsv(feature_path, col_names = F) %>% tidyr::unite(feature)
barcodes <- readr::read_tsv(barcode_path, col_names = F) %>% tidyr::unite(barcode)
 
mtx <- Matrix::readMM(mtx_path) %>%
  magrittr::set_rownames(features$feature) %>%
  magrittr::set_colnames(barcodes$barcode)
```

Including Reading Other Files

```R
require(magrittr)
require(readr)
require(Matrix)
require(tidyr)
require(dplyr)
library(Signac)
library(Seurat)

read_signac_C4 <- function(mex_dir_path, fragments, singlecellmetadata){
    mtx_path <- paste(mex_dir_path, "matrix.mtx.gz", sep = '/')
    feature_path <- paste(mex_dir_path, "peaks.bed.gz", sep = '/')
    barcode_path <- paste(mex_dir_path, "barcodes.tsv.gz", sep = '/')
    
    features <- readr::read_tsv(feature_path, col_names = F) %>% tidyr::unite(feature)
    barcodes <- readr::read_tsv(barcode_path, col_names = F) %>% tidyr::unite(barcode)
    
    mtx <- Matrix::readMM(mtx_path) %>%
    magrittr::set_rownames(features$feature) %>%
    magrittr::set_colnames(barcodes$barcode)

    metadata <- read.csv(
        file = singlecellmetadata,
        header = TRUE,
        row.names = 1
    )
    
    metadata$log10_uniqueFrags=log10(metadata$fragments)
    metadata$pct_reads_in_peaks <- metadata$peak_region_fragments / metadata$fragments * 100
    metadata$pct_reads_in_tss <- metadata$TSS_region_fragments / metadata$fragments * 100

    chrom_assay <- CreateChromatinAssay(
        counts = mtx,
        sep = c("_", "_"),
        fragments = fragments,
        min.cells = 10,
        min.features = 200
    )

    scATAC <- CreateSeuratObject(
        counts = chrom_assay,
        assay = "peaks",
        meta.data = metadata
    )
}
```



#### ArchR

```r
library(ArchR)
ArrowFiles <- createArrowFiles(
  inputFiles = FragmentFiles,
  sampleNames = names(inputFiles),
  filterTSS = 4, 
  filterFrags = 1000, 
  addTileMat = TRUE,
  addGeneScoreMat = TRUE
)
```



#### anndata

```R
import os
from scipy.sparse import csr_matrix
import anndata
import pandas as pd
import scipy.io
import scipy.sparse

def read_atac_C4(path): 
    file_list=os.listdir(path)
    for file in file_list:
        file_path = os.path.join(path, file)
        if file =="barcodes.tsv.gz":       
            obs = pd.read_csv(file_path, header=None, index_col=0, sep="\t")
            obs.index.name = ""
        elif file =="peaks.bed.gz":                   
            var = pd.read_csv(file_path, header=None, index_col=None, sep="\t")
            new_index = var[0].astype(str) + ':' + var[1].astype(str) + '-' + var[2].astype(str)
            var.index = new_index
            var.drop(columns=[0, 1, 2], inplace=True)
            var.index.name = ""
        elif file =="matrix.mtx.gz":    
            mtx = csr_matrix(scipy.io.mmread(file_path).T)
    adata=anndata.AnnData(mtx,obs=obs,var=var)
    adata.var_names_make_unique()
    return adata
```
