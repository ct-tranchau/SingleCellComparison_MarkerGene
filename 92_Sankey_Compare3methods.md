Common Marker genes among three detection methods
================
2022-12-06

## Loading the orthofinder output

``` r
# Clean ortholog find
orthofinder = read.csv("/Users/tranchau/Documents/SC_crossSpecies/OrthoFinder_source/Ath_maize_tom_rice/OrthoFinder/Results_Aug04/Orthogroups/Orthogroups.tsv", header = TRUE, sep = "\t")

library(tidyr); library(dplyr); library("stringr") ; library(reshape); library(ggplot2)
```

    ## Warning: package 'tidyr' was built under R version 4.1.2

    ## Warning: package 'dplyr' was built under R version 4.1.2

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    ## Warning: package 'reshape' was built under R version 4.1.2

    ## 
    ## Attaching package: 'reshape'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     rename

    ## The following objects are masked from 'package:tidyr':
    ## 
    ##     expand, smiths

    ## Warning: package 'ggplot2' was built under R version 4.1.2

``` r
og_ath = orthofinder[,c("Orthogroup", "Arabidopsis")] %>%
  mutate(Arabidopsis = strsplit(as.character(Arabidopsis), ",")) %>% # Split long string in a row into multiple rows
  unnest(Arabidopsis) %>%
  mutate(Arabidopsis = str_extract(Arabidopsis, "[^.]+"))  %>% # Extract all character before the first dot
  mutate(across(where(is.character), str_trim)) %>%  # Remove white spaces
  distinct(Arabidopsis, .keep_all = TRUE)  # Remove duplicated rows based on Arabidopsis column
  #remove_rownames %>% column_to_rownames(var="Arabidopsis")


og_maize = orthofinder[,c("Orthogroup", "Zeamays")] %>%
  mutate(Zeamays = strsplit(as.character(Zeamays), ",")) %>% # Split long string in a row into multiple rows
  unnest(Zeamays) %>%
  mutate(Zeamays = str_extract(Zeamays, "[^_]+")) %>% # Extract all character before the first underscore
  mutate(across(where(is.character), str_trim)) %>% # Remove white spaces
  distinct(Zeamays, .keep_all = TRUE)  # Remove duplicated rows based on Zeamays column
  #remove_rownames %>% column_to_rownames(var="Zeamays")


og_oryza = orthofinder[,c("Orthogroup", "Oryza")] %>%
  mutate(Oryza = strsplit(as.character(Oryza), ",")) %>% # Split long string in a row into multiple rows
  unnest(Oryza) %>%
  mutate(Oryza = str_extract(Oryza, "[^-]+")) %>% # Extract all character before the first underscore
  mutate(across(where(is.character), str_trim)) %>% # Remove white spaces
  distinct(Oryza, .keep_all = TRUE)  %>% # Remove duplicated rows based on Oryza column
  mutate(across('Oryza', str_replace, 't', 'g'))


og_tom = orthofinder[,c("Orthogroup", "Solanum")] %>%
  mutate(Solanum = strsplit(as.character(Solanum), ",")) %>% # Split long string in a row into multiple rows
  unnest(Solanum) %>%
  mutate(Solanum = str_extract(Solanum, "[^.]+"))  %>% # Extract all character before the first dot
  mutate(across(where(is.character), str_trim)) %>%  # Remove white spaces
  distinct(Solanum, .keep_all = TRUE)  # Remove duplicated rows based on Solanum column
  #remove_rownames %>% column_to_rownames(var="Solanum")
```

## Loading Seurat Marker genes

``` r
#---------------------------------------#---------------------------------------
MG_ara = readRDS("/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Data/MG_092522_Ath_05.RData")
#---------------------------------------
MG_maize = readRDS("/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Data/MG_092522_Maize_05.RData")
#---------------------------------------
MG_oryza = readRDS("/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Data/MG_092522_Rice_05.RData")
#---------------------------------------
MG_tom = readRDS("/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Data/MG_092522_Tom_05_110899.RData")
#---------------------------------------#---------------------------------------

# Merge marker gene and OG gene table
# Arabidopsis #######################
Ath_MG_OG_Seurat = merge(MG_ara, og_ath, by.x = "gene", by.y = "Arabidopsis") %>% arrange( cluster, desc(avg_log2FC)) %>% select("gene", "Orthogroup", "cluster", "avg_log2FC") %>% group_by(cluster) %>% top_n(200)
```

    ## Selecting by avg_log2FC

``` r
# Maize #############################
Maize_MG_OG_Seurat = merge(MG_maize, og_maize, by.x = "gene", by.y = "Zeamays") %>% arrange( cluster, desc(avg_log2FC)) %>% select("gene", "Orthogroup", "cluster", "avg_log2FC") %>% group_by(cluster) %>% top_n(200)
```

    ## Selecting by avg_log2FC

``` r
# Oryza #############################
Rice_MG_OG_Seurat = merge(MG_oryza, og_oryza, by.x = "gene", by.y = "Oryza") %>% arrange( cluster, desc(avg_log2FC)) %>% select("gene", "Orthogroup", "cluster", "avg_log2FC") %>% group_by(cluster) %>% top_n(200)
```

    ## Selecting by avg_log2FC

``` r
# Tomato #############################
Tom_MG_OG_Seurat = merge(MG_tom, og_tom, by.x = "gene", by.y = "Solanum") %>% arrange( cluster, desc(avg_log2FC)) %>% select("gene", "Orthogroup", "cluster", "avg_log2FC") %>% group_by(cluster) %>% top_n(200)
```

    ## Selecting by avg_log2FC

## Loading SHAP output marker genes

``` r
ATH_marker = read.table("/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Data_SPMarker/111022_ATH_out/opt_SHAP_markers_dir/opt_all_novel_marker.txt")
ATH_marker = ATH_marker[-which(ATH_marker$V1 == "feature"),]
ATH_marker$V4 = as.numeric(ATH_marker$V4)

Maize_marker = read.table("/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Data_SPMarker/111022_Maize_out/opt_SHAP_markers_dir/opt_all_novel_marker.txt")
Maize_marker = Maize_marker[-which(Maize_marker$V1 == "feature"),]
Maize_marker$V4 = as.numeric(Maize_marker$V4)

Rice_marker = read.table("/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Data_SPMarker/111022_Rice_out/opt_SHAP_markers_dir/opt_all_novel_marker.txt")
Rice_marker = Rice_marker[-which(Rice_marker$V1== "feature"),]
Rice_marker$V4 = as.numeric(Rice_marker$V4)


Tom_marker = read.table("/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Data_SPMarker/111022_Tom_out/opt_SHAP_markers_dir/opt_all_novel_marker.txt")
Tom_marker = Tom_marker[-which(Tom_marker$V1== "feature"),]
Tom_marker$V4 = as.numeric(Tom_marker$V4)

# Merge ortholog file and marker gene
ATH_MG_OG_SHAP = merge(ATH_marker, og_ath, by.x = "V1", by.y = "Arabidopsis") %>% arrange(V2, desc(V4)) %>% group_by(V2) %>% slice(1:200)

Maize_MG_OG_SHAP = merge(Maize_marker, og_maize, by.x = "V1", by.y = "Zeamays") %>% arrange(V2, desc(V4)) %>% group_by(V2) %>% slice(1:200)

Rice_MG_OG_SHAP = merge(Rice_marker, og_oryza, by.x = "V1", by.y = "Oryza") %>% arrange(V2, desc(V4)) %>% group_by(V2) %>% slice(1:200)

Tom_MG_OG_SHAP = merge(Tom_marker, og_tom, by.x = "V1", by.y = "Solanum") %>% arrange(V2, desc(V4)) %>% group_by(V2) %>% slice(1:200)
```

## Loading SVM marker genes

``` r
ATH_marker = read.table("/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Data_SPMarker/111022_ATH_out/opt_SVM_markers_dir/opt_all_novel_marker.txt")
ATH_marker = ATH_marker[-which(ATH_marker$V1 == "feature"),]
ATH_marker$V4 = as.numeric(ATH_marker$V4)

Maize_marker = read.table("/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Data_SPMarker/111022_Maize_out/opt_SVM_markers_dir/opt_all_novel_marker.txt")
Maize_marker = Maize_marker[-which(Maize_marker$V1 == "feature"),]
Maize_marker$V4 = as.numeric(Maize_marker$V4)

Rice_marker = read.table("/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Data_SPMarker/111022_Rice_out/opt_SVM_markers_dir/opt_all_novel_marker.txt")
Rice_marker = Rice_marker[-which(Rice_marker$V1== "feature"),]
Rice_marker$V4 = as.numeric(Rice_marker$V4)

Tom_marker = read.table("/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Data_SPMarker/111022_Tom_out/opt_SVM_markers_dir/opt_all_novel_marker.txt")
Tom_marker = Tom_marker[-which(Tom_marker$V1== "feature"),]
Tom_marker$V4 = as.numeric(Tom_marker$V4)


ATH_MG_OG_SVM = merge(ATH_marker, og_ath, by.x = "V1", by.y = "Arabidopsis") %>% arrange(V2, desc(V4)) %>% group_by(V2) %>% slice(1:200)

Maize_MG_OG_SVM = merge(Maize_marker, og_maize, by.x = "V1", by.y = "Zeamays") %>% arrange(V2, desc(V4)) %>% group_by(V2) %>% slice(1:200)

Rice_MG_OG_SVM = merge(Rice_marker, og_oryza, by.x = "V1", by.y = "Oryza") %>% arrange(V2, desc(V4)) %>% group_by(V2) %>% slice(1:200)

Tom_MG_OG_SVM = merge(Tom_marker, og_tom, by.x = "V1", by.y = "Solanum") %>% arrange(V2, desc(V4)) %>% group_by(V2) %>% slice(1:200)
```

## Common Marker genes among three methods in Arabidopsis data

``` r
short_Ath_MG_OG_Seurat = Ath_MG_OG_Seurat[Ath_MG_OG_Seurat$cluster %in% intersect(unique(ATH_MG_OG_SHAP$V2), unique(ATH_MG_OG_SVM$V2)), ]
colnames(short_Ath_MG_OG_Seurat) = c("V1", "Orthogroup", "V2", "V4")
#short_Ath_MG_OG_Seurat
#ATH_MG_OG_SHAP
#ATH_MG_OG_SVM

count_com_OG = function(Species1, Species2){
  clusters_S1 = unique(Species1$V2); clusters_S2 = unique(Species2$V2)
  two_plants <- matrix(nrow=length(clusters_S1), ncol=length(clusters_S2))
  for(i in 1:length(clusters_S1)){
    for(j in 1:length(clusters_S2)){
      list_overlap = intersect(Species1[Species1$V2 == clusters_S1[i],]$V1, Species2[Species2$V2 == clusters_S2[j],]$V1)
      num_overlap = length(list_overlap)
    
      two_plants[i,j] = num_overlap
    }
  }
  rownames(two_plants) = unique(Species1$V2)
  colnames(two_plants) = unique(Species2$V2) 
  return(two_plants[order(rownames(two_plants)), order(colnames(two_plants))])
}

ATH_Seurat_SHAP = count_com_OG(short_Ath_MG_OG_Seurat, ATH_MG_OG_SHAP) %>% melt()
```

    ## Warning in type.convert.default(X[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(X[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

``` r
ATH_Seurat_SHAP$X1 = paste(ATH_Seurat_SHAP$X1, "Seurat", sep="_")
ATH_Seurat_SHAP$X2 = paste(ATH_Seurat_SHAP$X2, "SHAP", sep="_")
ATH_SHAP_SVM = count_com_OG(ATH_MG_OG_SHAP, ATH_MG_OG_SVM) %>% melt()
```

    ## Warning in type.convert.default(X[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(X[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

``` r
ATH_SHAP_SVM$X1 = paste(ATH_SHAP_SVM$X1, "SHAP", sep="_")
ATH_SHAP_SVM$X2 = paste(ATH_SHAP_SVM$X2, "SVM", sep="_")
data_long = rbind(ATH_Seurat_SHAP, ATH_SHAP_SVM) %>% filter(value > 0)


# Require packages for Sankey diagrams
library(tidyverse)
```

    ## ?????? Attaching packages ????????????????????????????????????????????????????????????????????????????????????????????????????????????????????? tidyverse 1.3.1 ??????

    ## ??? tibble  3.1.7     ??? purrr   0.3.4
    ## ??? readr   2.1.2     ??? forcats 0.5.1

    ## Warning: package 'tibble' was built under R version 4.1.2

    ## Warning: package 'readr' was built under R version 4.1.2

    ## ?????? Conflicts ?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????? tidyverse_conflicts() ??????
    ## ??? reshape::expand() masks tidyr::expand()
    ## ??? dplyr::filter()   masks stats::filter()
    ## ??? dplyr::lag()      masks stats::lag()
    ## ??? reshape::rename() masks dplyr::rename()

``` r
library(viridis)
```

    ## Loading required package: viridisLite

``` r
library(patchwork)
library(hrbrthemes)
```

    ## NOTE: Either Arial Narrow or Roboto Condensed fonts are required to use these themes.

    ##       Please use hrbrthemes::import_roboto_condensed() to install Roboto Condensed and

    ##       if Arial Narrow is not on your system, please see https://bit.ly/arialnarrow

``` r
library(circlize)
```

    ## Warning: package 'circlize' was built under R version 4.1.2

    ## ========================================
    ## circlize version 0.4.15
    ## CRAN page: https://cran.r-project.org/package=circlize
    ## Github page: https://github.com/jokergoo/circlize
    ## Documentation: https://jokergoo.github.io/circlize_book/book/
    ## 
    ## If you use it in published research, please cite:
    ## Gu, Z. circlize implements and enhances circular visualization
    ##   in R. Bioinformatics 2014.
    ## 
    ## This message can be suppressed by:
    ##   suppressPackageStartupMessages(library(circlize))
    ## ========================================

``` r
library(networkD3)



colnames(data_long) <- c("source", "target", "value")


# From these flows we need to create a node data frame: it lists every entities involved in the flow
nodes <- data.frame(name=c(as.character(data_long$source), as.character(data_long$target)) %>% unique())

data_long$IDsource=match(data_long$source, nodes$name)-1 
data_long$IDtarget=match(data_long$target, nodes$name)-1

# prepare colour scale
my_color <- 'd3.scaleOrdinal() .domain(["Atrichoblast1_Seurat", "Atrichoblast2_Seurat", "Cortex_Seurat", "Endodermis_Seurat", "Meristem_Endocortex_Seurat", "Phloem_Seurat", "Stele1_Seurat", "Stele2_Seurat", "Trichoblast1_Seurat", "Trichoblast2_Seurat", "Xylem_Seurat", "Atrichoblast1_SHAP", "Atrichoblast2_SHAP", "Cortex_SHAP", "Endodermis_SHAP", "Meristem_Endocortex_SHAP", "Phloem_SHAP", "Stele1_SHAP", "Stele2_SHAP", "Trichoblast1_SHAP", "Trichoblast2_SHAP", "Xylem_SHAP", "Atrichoblast1_SVM", "Atrichoblast2_SVM", "Cortex_SVM", "Endodermis_SVM", "Meristem_Endocortex_SVM", "Phloem_SVM", "Stele1_SVM", "Stele2_SVM", "Trichoblast1_SVM", "Trichoblast2_SVM", "Xylem_SVM"]) .range(["yellow", "lightgreen", "orange", "purple", "lightblue", "red", "orangered", "steelblue", "green", "hotpink", "darkblue"])'

# Make the Network
san_ath = sankeyNetwork(Links = data_long, Nodes = nodes,
                     Source = "IDsource", Target = "IDtarget",
                     Value = "value", NodeID = "name", 
                     sinksRight=FALSE,  nodeWidth= 40, fontSize=20, nodePadding=20, colourScale=my_color)
san_ath
```

![](92_Sankey_Compare3methods_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

``` r
library(rbokeh)
#saveNetwork(san_ath, "/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Figure/F3_sankey/ATH_sankey.html")

library(webshot)
```

    ## Warning: package 'webshot' was built under R version 4.1.2

``` r
#webshot("/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Figure/F3_sankey/ATH_sankey.html","/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Figure/F3_sankey/ATH_sankey.png", vwidth = 1800, vheight = 1000)
```

## Common Marker Genes among three detection methods in Maize data

``` r
short_Maize_MG_OG_Seurat = Maize_MG_OG_Seurat[Maize_MG_OG_Seurat$cluster %in% intersect(unique(Maize_MG_OG_SHAP$V2), unique(Maize_MG_OG_SVM$V2)), ]
colnames(short_Maize_MG_OG_Seurat) = c("V1", "Orthogroup", "V2", "V4")
#short_Maize_MG_OG_Seurat
#Maize_MG_OG_SHAP
#Maize_MG_OG_SVM

count_com_OG = function(Species1, Species2){
  clusters_S1 = unique(Species1$V2); clusters_S2 = unique(Species2$V2)
  two_plants <- matrix(nrow=length(clusters_S1), ncol=length(clusters_S2))
  for(i in 1:length(clusters_S1)){
    for(j in 1:length(clusters_S2)){
      list_overlap = intersect(Species1[Species1$V2 == clusters_S1[i],]$V1, Species2[Species2$V2 == clusters_S2[j],]$V1)
      num_overlap = length(list_overlap)
    
      two_plants[i,j] = num_overlap
    }
  }
  rownames(two_plants) = unique(Species1$V2)
  colnames(two_plants) = unique(Species2$V2) 
  return(two_plants[order(rownames(two_plants)), order(colnames(two_plants))])
}

Maize_Seurat_SHAP = count_com_OG(short_Maize_MG_OG_Seurat, Maize_MG_OG_SHAP) %>% melt()
```

    ## Warning in type.convert.default(X[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(X[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

``` r
Maize_Seurat_SHAP$X1 = paste(Maize_Seurat_SHAP$X1, "Seurat", sep="_")
Maize_Seurat_SHAP$X2 = paste(Maize_Seurat_SHAP$X2, "SHAP", sep="_")
Maize_SHAP_SVM = count_com_OG(Maize_MG_OG_SHAP, Maize_MG_OG_SVM) %>% melt()
```

    ## Warning in type.convert.default(X[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(X[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

``` r
Maize_SHAP_SVM$X1 = paste(Maize_SHAP_SVM$X1, "SHAP", sep="_")
Maize_SHAP_SVM$X2 = paste(Maize_SHAP_SVM$X2, "SVM", sep="_")
data_long = rbind(Maize_Seurat_SHAP, Maize_SHAP_SVM) %>% filter(value > 0)

colnames(data_long) <- c("source", "target", "value")
###################3
# From these flows we need to create a node data frame: it lists every entities involved in the flow
nodes <- data.frame(name=c(as.character(data_long$source), as.character(data_long$target)) %>% unique())
data_long$IDsource=match(data_long$source, nodes$name)-1 
data_long$IDtarget=match(data_long$target, nodes$name)-1

# prepare colour scale
#my_color <- 'd3.scaleOrdinal() .domain(["Atrichoblast1_Seurat", "Atrichoblast2_Seurat", "Cortex_Seurat", "Endodermis_Seurat", "Meristem_Endocortex_Seurat", "Phloem_Seurat", "Stele1_Seurat", "Stele2_Seurat", "Trichoblast1_Seurat", "Trichoblast2_Seurat", "Xylem_Seurat", "Atrichoblast1_SHAP", "Atrichoblast2_SHAP", "Cortex_SHAP", "Endodermis_SHAP", "Meristem_Endocortex_SHAP", "Phloem_SHAP", "Stele1_SHAP", "Stele2_SHAP", "Trichoblast1_SHAP", "Trichoblast2_SHAP", "Xylem_SHAP", "Atrichoblast1_SVM", "Atrichoblast2_SVM", "Cortex_SVM", "Endodermis_SVM", "Meristem_Endocortex_SVM", "Phloem_SVM", "Stele1_SVM", "Stele2_SVM", "Trichoblast1_SVM", "Trichoblast2_SVM", "Xylem_SVM"]) .range(["yellow", "lightgreen", "orange", "purple", "lightblue", "red", "orangered", "steelblue", "green", "hotpink", "darkblue"])'

# Make the Network
san_ath = sankeyNetwork(Links = data_long, Nodes = nodes,
                     Source = "IDsource", Target = "IDtarget",
                     Value = "value", NodeID = "name", 
                     sinksRight=FALSE,  nodeWidth= 40, fontSize=20, nodePadding=20) #, colourScale=my_color)
san_ath
```

![](92_Sankey_Compare3methods_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

``` r
#### Saving
library(rbokeh)
#saveNetwork(san_ath, "/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Figure/F3_sankey/Maize_sankey.html")

library(webshot)
#webshot("/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Figure/F3_sankey/Maize_sankey.html","/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Figure/F3_sankey/Maize_sankey.png", vwidth = 1800, vheight = 1000)
```

## Sankey for Rice dataset; Common Marker genes among three methods

``` r
short_Rice_MG_OG_Seurat = Rice_MG_OG_Seurat[Rice_MG_OG_Seurat$cluster %in% intersect(unique(Rice_MG_OG_SHAP$V2), unique(Rice_MG_OG_SVM$V2)), ]
colnames(short_Rice_MG_OG_Seurat) = c("V1", "Orthogroup", "V2", "V4")
#short_Rice_MG_OG_Seurat
#Rice_MG_OG_SHAP
#Rice_MG_OG_SVM

count_com_OG = function(Species1, Species2){
  clusters_S1 = unique(Species1$V2); clusters_S2 = unique(Species2$V2)
  two_plants <- matrix(nrow=length(clusters_S1), ncol=length(clusters_S2))
  for(i in 1:length(clusters_S1)){
    for(j in 1:length(clusters_S2)){
      list_overlap = intersect(Species1[Species1$V2 == clusters_S1[i],]$V1, Species2[Species2$V2 == clusters_S2[j],]$V1)
      num_overlap = length(list_overlap)
    
      two_plants[i,j] = num_overlap
    }
  }
  rownames(two_plants) = unique(Species1$V2)
  colnames(two_plants) = unique(Species2$V2) 
  return(two_plants[order(rownames(two_plants)), order(colnames(two_plants))])
}

Rice_Seurat_SHAP = count_com_OG(short_Rice_MG_OG_Seurat, Rice_MG_OG_SHAP) %>% melt()
```

    ## Warning in type.convert.default(X[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(X[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

``` r
Rice_Seurat_SHAP$X1 = paste(Rice_Seurat_SHAP$X1, "Seurat", sep="_")
Rice_Seurat_SHAP$X2 = paste(Rice_Seurat_SHAP$X2, "SHAP", sep="_")
Rice_SHAP_SVM = count_com_OG(Rice_MG_OG_SHAP, Rice_MG_OG_SVM) %>% melt()
```

    ## Warning in type.convert.default(X[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(X[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

``` r
Rice_SHAP_SVM$X1 = paste(Rice_SHAP_SVM$X1, "SHAP", sep="_")
Rice_SHAP_SVM$X2 = paste(Rice_SHAP_SVM$X2, "SVM", sep="_")
data_long = rbind(Rice_Seurat_SHAP, Rice_SHAP_SVM) %>% filter(value > 0)

colnames(data_long) <- c("source", "target", "value")
###################3
# From these flows we need to create a node data frame: it lists every entities involved in the flow
nodes <- data.frame(name=c(as.character(data_long$source), as.character(data_long$target)) %>% unique())
data_long$IDsource=match(data_long$source, nodes$name)-1 
data_long$IDtarget=match(data_long$target, nodes$name)-1

# prepare colour scale
my_color <- 'd3.scaleOrdinal() .domain(["Atrichoblast_Seurat", "Cortex_Seurat", "Cortex_like_Seurat", "Exodermis_Seurat", "Phloem_Seurat", "Stele_Seurat", "Trichoblast1_Seurat", "Trichoblast2_Seurat", "Xylem_Seurat", "Endodermis_Seurat", "Meristem_Seurat", "Atrichoblast_SHAP", "Cortex_SHAP", "Cortex_like_SHAP", "Exodermis_SHAP", "Phloem_SHAP", "Stele_SHAP", "Trichoblast1_SHAP", "Trichoblast2_SHAP", "Xylem_SHAP", "Endodermis_SHAP", "Meristem_SHAP", "Atrichoblast_SVM", "Cortex_SVM", "Cortex_like_SVM", "Exodermis_SVM", "Phloem_SVM", "Stele_SVM", "Trichoblast1_SVM", "Trichoblast2_SVM", "Xylem_SVM", "Endodermis_SVM", "Meristem_SVM"]) .range(["yellow", "lightgreen", "orange", "purple", "lightblue", "red", "orangered", "steelblue", "green", "hotpink", "darkblue"])'

# Make the Network
san_ath = sankeyNetwork(Links = data_long, Nodes = nodes,
                     Source = "IDsource", Target = "IDtarget",
                     Value = "value", NodeID = "name", 
                     sinksRight=FALSE,  nodeWidth= 40, fontSize=20, nodePadding=20, colourScale=my_color)
san_ath
```

![](92_Sankey_Compare3methods_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

``` r
#### Saving
library(rbokeh)
#saveNetwork(san_ath, "/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Figure/F3_sankey/Rice_sankey.html")

library(webshot)
#webshot("/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Figure/F3_sankey/Rice_sankey.html","/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Figure/F3_sankey/Rice_sankey.png", vwidth = 1800, vheight = 1000)
```

## Common Marker genes among three methods in Tomato data

``` r
short_Tom_MG_OG_Seurat = Tom_MG_OG_Seurat[Tom_MG_OG_Seurat$cluster %in% intersect(unique(Tom_MG_OG_SHAP$V2), unique(Tom_MG_OG_SVM$V2)), ]
colnames(short_Tom_MG_OG_Seurat) = c("V1", "Orthogroup", "V2", "V4")
#short_Tom_MG_OG_Seurat
#Tom_MG_OG_SHAP
#Tom_MG_OG_SVM

count_com_OG = function(Species1, Species2){
  clusters_S1 = unique(Species1$V2); clusters_S2 = unique(Species2$V2)
  two_plants <- matrix(nrow=length(clusters_S1), ncol=length(clusters_S2))
  for(i in 1:length(clusters_S1)){
    for(j in 1:length(clusters_S2)){
      list_overlap = intersect(Species1[Species1$V2 == clusters_S1[i],]$V1, Species2[Species2$V2 == clusters_S2[j],]$V1)
      num_overlap = length(list_overlap)
    
      two_plants[i,j] = num_overlap
    }
  }
  rownames(two_plants) = unique(Species1$V2)
  colnames(two_plants) = unique(Species2$V2) 
  return(two_plants[order(rownames(two_plants)), order(colnames(two_plants))])
}

Tom_Seurat_SHAP = count_com_OG(short_Tom_MG_OG_Seurat, Tom_MG_OG_SHAP) %>% melt()
```

    ## Warning in type.convert.default(X[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(X[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

``` r
Tom_Seurat_SHAP$X1 = paste(Tom_Seurat_SHAP$X1, "Seurat", sep="_")
Tom_Seurat_SHAP$X2 = paste(Tom_Seurat_SHAP$X2, "SHAP", sep="_")
Tom_SHAP_SVM = count_com_OG(Tom_MG_OG_SHAP, Tom_MG_OG_SVM) %>% melt()
```

    ## Warning in type.convert.default(X[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

    ## Warning in type.convert.default(X[[i]], ...): 'as.is' should be specified by the
    ## caller; using TRUE

``` r
Tom_SHAP_SVM$X1 = paste(Tom_SHAP_SVM$X1, "SHAP", sep="_")
Tom_SHAP_SVM$X2 = paste(Tom_SHAP_SVM$X2, "SVM", sep="_")
data_long = rbind(Tom_Seurat_SHAP, Tom_SHAP_SVM) %>% filter(value > 0)

colnames(data_long) <- c("source", "target", "value")
###################3
# From these flows we need to create a node data frame: it lists every entities involved in the flow
nodes <- data.frame(name=c(as.character(data_long$source), as.character(data_long$target)) %>% unique())
data_long$IDsource=match(data_long$source, nodes$name)-1 
data_long$IDtarget=match(data_long$target, nodes$name)-1

# prepare colour scale
my_color <- 'd3.scaleOrdinal() .domain(["Epidermis1_Seurat", "Epidermis2_Seurat", "EXO_iCOR1_Seurat" , "EXO_iCOR2_Seurat" , "Exodermis1_Seurat", "Exodermis2_Seurat", "Phloem2_Seurat", "Vascular2_Seurat", "Xylem_Seurat", "Phloem1_Seurat", "Vascular1_Seurat", "MZ_Seurat", "Epidermis1_SHAP", "Epidermis2_SHAP", "EXO_iCOR1_SHAP" , "EXO_iCOR2_SHAP" , "Exodermis1_SHAP", "Exodermis2_SHAP", "Phloem2_SHAP", "Vascular2_SHAP", "Xylem_SHAP", "Phloem1_SHAP", "Vascular1_SHAP", "MZ_SHAP", "Epidermis1_SVM", "Epidermis2_SVM", "EXO_iCOR1_SVM" , "EXO_iCOR2_SVM" , "Exodermis1_SVM", "Exodermis2_SVM", "Phloem2_SVM", "Vascular2_SVM", "Xylem_SVM", "Phloem1_SVM", "Vascular1_SVM", "MZ_SVM"]) .range(["yellow", "lightgreen", "orange", "purple", "lightblue", "red", "orangered", "steelblue", "green", "hotpink", "darkblue", "black"])'

# Make the Network
san_ath = sankeyNetwork(Links = data_long, Nodes = nodes,
                     Source = "IDsource", Target = "IDtarget",
                     Value = "value", NodeID = "name", 
                     sinksRight=FALSE,  nodeWidth= 40, fontSize=20, nodePadding=20 , colourScale=my_color)
san_ath
```

![](92_Sankey_Compare3methods_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

``` r
#### Saving
library(rbokeh)
#saveNetwork(san_ath, "/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Figure/F3_sankey/Tom_sankey.html")

library(webshot)
#webshot("/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Figure/F3_sankey/Tom_sankey.html","/Users/tranchau/Documents/SC_crossSpecies/092522_allSteps/Figure/F3_sankey/Tom_sankey.png", vwidth = 1800, vheight = 1000)
```
