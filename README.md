
<!-- README.md is generated from README.Rmd. Please edit that file -->

# CACIMAR

<!-- badges: start -->
<!-- badges: end -->

CACIMAR is an R package to identify cross-species marker genes, cell
types and gene regulatory networks based on scRNA-seq data.

## Installation

Install CACIMAR from github, run:

``` r
# install.packages("devtools")
devtools::install_github("jiang-junyao/CACIMAR")
```

## Inputs data

### (1) Seurat object

Seurat object should have clustering information in active.ident slot
and meta.data slot

### (2) Marker genes table (used for identifying cell types)

Rownames of Marker genes table should be the same format as the rownames
format of seurat object, and should contain CellType column (Marker
genes table can also contain other columns, which will not affect this
function)

## Tutorial

### 1.Identify Cell types

Using known marker genes to annotate each cluster. This method is based
on AUC (area under the receiver operating characteristic curve of gene
expression), and is very sensitive to the marker genes input.

``` r
library(CACIMAR)
Marker <- read.table('D:\\GIBH\\platform\\test data/Retinal_markersZf.txt',header = T)
head(Marker)
#>                     Symbol            CellType
#> ENSDARG00000045904   nr2e3 Rods,Rodprogenitors
#> ENSDARG00000019566 neurod1      RodProgenitors
#> ENSDARG00000099572   hmgn2      RodProgenitors
#> ENSDARG00000002193     rho                Rods
#> ENSDARG00000100466     nrl                Rods
#> ENSDARG00000011235    otx2                Rods
```

``` r
### I identify cell type for 3 clusters here to reduce the running time
seurat_object <- readRDS('D:\\GIBH\\platform\\test data/Zebrafishdata.rds')
seurat_object <- subset(seurat_object,idents = c(1,2,3))
zfcelltype <- Identify_CellType(seurat_object,Marker)
```

### 2.Identify markers

In this part, CACIMAR first uses ROC analysis in ‘FindAllMarkers’
function of
[Seurat](https://satijalab.org/seurat/articles/get_started.html) to
identify marker genes in each cluster. Then, based on marker genes
identified above, CACIMAR calculates the power of marker gene in each
and differences of marker gene between clusters, and marker genes with
high differences between clusters will be retained. Finally, CACIMAR
uses fisher test to identify significant cluster related to this marker
gene (p.value &lt;0.05).

``` r
Marker1 <- Identify_Markers(seurat_object,Spec1='Zf')
```

### 3.Identify cross-species marker genes

#### Identify cross-species marker genes in two species

In this part, CACIMAR first uses **ortholog genes database of two
species** to refine marker genes, marker genes in orthologs database
will be retained. Then, CACIMAR selects marker genes that are in the
same cell type of two species as cross-species marker genes of two
species.

``` r
###Get_Used_OrthG
Mm_marker_cell_type <- read.delim2("D:/GIBH/platform/test data/Mm_marker_cell_type.txt")
head(Mm_marker_cell_type)###This table must contain 'CellType' column
#>                    Symbol          CellType
#> ENSMUSG00000070348  Ccnd1           RPCs,MG
#> ENSMUSG00000006728   Cdk4              RPCs
#> ENSMUSG00000027168   Pax6 RPCs,MG,HC,AC,RGC
#> ENSMUSG00000021239   Vsx2 PrimaryPRCs,BC,MG
#> ENSMUSG00000000247   Lhx2    PrimaryPRCs,MG
#> ENSMUSG00000031073  Fgf15       PrimaryPRCs
Zf_marker_cell_type <- read.delim2("D:/GIBH/platform/test data/Zf_marker_cell_type.txt")
OrthG <- read.delim('D:/GIBH/platform/test data/RNA_genes_mmVSzf.txt')
ShMarker <- OrthG_TwoSpecies(OrthG,Mm_marker_cell_type,Zf_marker_cell_type,Species_name = c('mm','zf'))
#> 
#> mm_zf_0T1 mm_zf_1T0 mm_zf_1T1 mm_zf_1TN mm_zf_NT1 mm_zf_NTN 
#>     14207     37432     10353      3238       273       128 
#> [1] "mm_zf_0T1"
#> [1] "mm_zf_1T0"
#> [1] "mm_zf_1T1"
#> [1] "mm_zf_1TN"
#> [1] "mm_zf_NT1"
#> [1] "mm_zf_NTN"
```

#### Identify cross-species marker genes in three species

``` r
refined_markers<-Refine_ThreeSpecies(ShMarker,mmCelltype,zfCelltype,Species = c('mm','zf'))
```

Unkown codes

``` r
###usage???
###Get_Wilcox_Markers_Cond ###???usage???
wilcox<-read.table('D:\\GIBH\\platform\\test data/mmP60RmmNMDA_mmP60mmLD_wilcoxMG_MarkerGenes.txt')
Cor<-read.table('D:\\GIBH\\platform\\test data/mmP60RmmNMDA_mmLD_pbmcSubC_MG_Bin50_R5_GeneCor.txt',header = T)
wilcox_result<-get_Wilcox_Markers_Cond(wilcox,Cor)
###Overlap_Markers_Cond
mmMarkers3_F3F0 <- read.delim("D:/GIBH/platform/test data/mmP60RmmNMDA_mmP60mmLD_P03_Markers3_F3F0.txt")
zfMarkers3_F3F0 <- read.delim("D:/GIBH/platform/test data/zfAdzfNMDA_zfAdzfLD_zfAdzfTR_P03_Markers3_F3F0.txt")
mmCelltype<-read.table('D:/GIBH/platform/test data/mmP60RmmNMDA_mmP60mmLD_Cell_Types.txt',header = T)
zfCelltype<-read.table('D:/GIBH/platform/test data/zfAdzfNMDA_zfAdzfLD_zfAdzfTR_Cell_Types.txt',header = T)
mmMarker<-Overlap_Markers_Cond(mmMarkers3_F3F0,mmCelltype,Spec1='mm')
zfMarker<-Overlap_Markers_Cond(zfMarkers3_F3F0,zfCelltype,Spec1='zf')
```

### 4.plot MarkersHeaptmap

``` r
Marker1_plot<-Format_Markers_Frac(Marker1)
plot_MarkersHeatmap(Marker1_plot)
```

### 5.Cross-species celltype heamtmap

``` r
expression<-read.table('D:\\GIBH\\platform\\CellType_Comp\\CellType_Comp\\Data/mmP60RmmNMDA_chP10chNMDA_zfAdzfNMDA_Power01_SharedMarkers_Frac.txt')
celltypes<-read.delim("D:/GIBH/platform/CellType_Comp/CellType_Comp/Data/mmP60RmmNMDA_chP10chNMDA_zfAdzfNMDA_Cell_Types.txt")
a<-Heatmap_Cor(expression,celltypes,cluster_cols=T, cluster_rows=F)
Plot_tree(a)
```

\#\#\#6. Cross-species regulatory networks
