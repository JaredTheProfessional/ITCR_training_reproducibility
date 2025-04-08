# README
# Acute Myeloid Leukemia Heatmap

**Author:** CCDL for ALSF - Adapted for this repository by Candace Savonen  
**Date:** October 2021

This analysis has been adapted from this [refine.bio-examples notebook](https://alexslemonade.github.io/refinebio-examples/03-rnaseq/clustering_rnaseq_01_heatmap.html).

## Purpose of the Analysis

Making a change to have a non-merged pull request

In this analysis, we use an [acute myeloid leukemia sample dataset](https://www.refine.bio/experiments/SRP070849) from [Shih et al., 2017](https://pubmed.ncbi.nlm.nih.gov/28193779/) pre-processed by [refinebio](https://www.refine.bio/). The dataset contains RNA-sequencing results for 19 samples from acute myeloid leukemia (AML) model mice under controlled treatment conditions.

## Steps Involved

1. **Set up analysis folders**: Create directories for data, plots, and results.
2. **Install libraries**: Install and attach necessary R libraries (`pheatmap`, `magrittr`).
3. **Import and set up data**: Read in metadata and gene expression data.
4. **Choose genes of interest**: Filter genes based on variance.
5. **Prepare metadata for annotation**: Annotate the data with mutation and treatment information.
6. **Create annotated heatmap**: Generate and save a heatmap with annotations.
7. **Session info**: Print session information for reproducibility.

## Setup Instructions

1. **Create directories**:
        ```r
        if (!dir.exists("data")) {
            dir.create("data")
        }
        if (!dir.exists("plots")) {
            dir.create("plots")
        }
        if (!dir.exists("results")) {
            dir.create("results")
        }
        ```

2. **Install and attach libraries**:
        ```r
        if (!("pheatmap" %in% installed.packages())) {
            install.packages("pheatmap", update = FALSE)
        }
        library(pheatmap)
        library(magrittr)
        set.seed(12345)
        ```

3. **Import data**:
        ```r
        metadata <- readr::read_tsv(metadata_file)
        expression_df <- readr::read_tsv(data_file) %>%
            tibble::column_to_rownames("Gene")
        ```

4. **Filter genes by variance**:
        ```r
        variances <- apply(expression_df, 1, var)
        upper_var <- quantile(variances, 0.75)
        df_by_var <- data.frame(expression_df) %>%
            dplyr::filter(variances > upper_var)
        readr::write_tsv(df_by_var, file.path(results_dir, "top_90_var_genes.tsv"))
        ```

5. **Prepare metadata for annotation**:
        ```r
        annotation_df <- metadata %>%
            dplyr::mutate(
                mutation = dplyr::case_when(
                    startsWith(refinebio_title, "TET2") ~ "TET2",
                    startsWith(refinebio_title, "IDH2") ~ "IDH2",
                    startsWith(refinebio_title, "WT") ~ "WT",
                    TRUE ~ "unknown"
                )
            ) %>%
            dplyr::select(refinebio_accession_code, mutation, refinebio_treatment) %>%
            tibble::column_to_rownames("refinebio_accession_code")
        ```

6. **Create and save heatmap**:
        ```r
        heatmap_annotated <- pheatmap(
            df_by_var,
            cluster_rows = TRUE,
            cluster_cols = TRUE,
            show_rownames = FALSE,
            annotation_col = annotation_df,
            main = "Annotated Heatmap",
            colorRampPalette(c("deepskyblue", "black", "yellow"))(25),
            scale = "row"
        )
        png(file.path(plots_dir, "aml_heatmap.png"))
        heatmap_annotated
        dev.off()
        ```

7. **Print session info**:
        ```r
        sessioninfo::session_info()
        ```

## Resources

- [refine.bio-examples notebook](https://alexslemonade.github.io/refinebio-examples/03-rnaseq/clustering_rnaseq_01_heatmap.html)
- [Acute Myeloid Leukemia Sample Dataset](https://www.refine.bio/experiments/SRP070849)
- [Shih et al., 2017](https://pubmed.ncbi.nlm.nih.gov/28193779/)
- [refinebio](https://www.refine.bio/)