# zebrafish_variant_prioritization

## Overview

The zebrafish variant prioritization (zvp) pipeline is designed to discover, classify, and prioritize variants arising from forward genetic screens in danio rerio. The specific experimental design this pipeline was built for is described in https://www.sciencedirect.com/science/article/pii/S0012160620303079 and is for homozygous recessive variants in F3 progeny. More generally, zvp is expected to work for an incross of individuals heterozygous for the same genotypic variant, but it benefits from the epecific experimental design it was built for due to increased homozygosity around the variant of interest due to crossing-over in meiosis.

## Background and General strategy

In a heavily mutagenized line (such as occurs with ENU mutagensis), phenotypic progeny will carry many induced mutations as well as naturally occuring variants with respect to the published zebrafish genomic sequence (GRCz11 for this work). Whole genome sequencing (WGS) or whole exome sequencing (WES) and subsequent variant calling may identify millions or tens of millions of singlue nucleotide variants (SNVs) and small insertions and deletions (indels) in mutagenized lines. The goal of zvp is to greatly reduce the number of SNVs and indels that must be considered by experiemental scientists into a managable number while retaining and prioritizing potential causal candidates. zvp achieves this goal through knowledge of the experimental design to filter potential variants and through variant classification by inferred trascriptomic effects boosted ensembl VEP to score coding variants in terms of predicted effect. Phenotypic F3 fish are sequenced and their phenotyipcally WT sibglings (genotypically heterozygous and WT homozygous) are sequenced in parallel using either WGS or WES.

The general strategy of zvp is as follows:
1.) Filter out low confidence mutant variants.
2.) Filter out all variants found in dbSNP - we are looking for novel variants.
3.) Filter out all variants with the same allele frequency in both mutant and siblings - we expect mutants to homozygous recessive and siblings to be heterozygous with a 2:1 wt:mutant allele count.
4.)  

## Usage

zvp is distributed as a singularity image. This can be downloaded via:
```
wget http://regmedsrv1.wustl.edu/Public_SPACE/pgontarz/Public_html/Zebrafish_variant_calling_and_prioritization/zvp.simg
```

The singularity image is somewhat large but contains all necessary reference, customized annotation, and customized ensembl VEP database files to run locally. zvp is for use with GRCz11 as assemblies GRCz10 and zv9 are not supported by VEP. zvp
