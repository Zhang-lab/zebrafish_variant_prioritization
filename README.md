# zebrafish_variant_prioritization

## Overview

The zebrafish variant prioritization (zvp) pipeline is designed to discover, classify, and prioritize variants arising from forward genetic screens in danio rerio. The specific experimental design this pipeline was built for is described in https://www.sciencedirect.com/science/article/pii/S0012160620303079 and is for homozygous recessive variants in F3 progeny. More generally, zvp is expected to work for an incross of individuals heterozygous for the same genotypic variant, but it benefits from the specific experimental design it was built for due to increased homozygosity around the variant of interest due to crossing-over in meiosis.

## Background and General strategy

In a heavily mutagenized line (such as occurs with ENU mutagensis), phenotypic progeny will carry many induced mutations as well as naturally occuring variants with respect to the published zebrafish genomic sequence (GRCz11 for this work). Whole genome sequencing (WGS) or whole exome sequencing (WES) and subsequent variant calling may identify millions or tens of millions of singlue nucleotide variants (SNVs) and small insertions and deletions (indels) in mutagenized lines. The goal of zvp is to greatly reduce the number of SNVs and indels that must be considered by experiemental scientists into a managable number while retaining and prioritizing potential causal candidates. zvp achieves this goal through knowledge of the experimental design to filter potential variants and through variant classification by inferred trascriptomic effects boosted ensembl VEP to score coding variants in terms of predicted effect. Phenotypic F3 fish are sequenced and their phenotyipcally WT sibglings (genotypically heterozygous and WT homozygous) are sequenced in parallel using either WGS or WES.

The general strategy of zvp is as follows:
1) Filter out low confidence mutant variants.
2) Filter out all variants found in dbSNP - we are looking for novel variants!
3) Filter out all variants with the same allele frequency in both mutant and siblings - we expect mutants to homozygous recessive and siblings to be heterozygous with a 2:1 wt:mutant allele count.
4) Separate filtered variants into three regions: Coding regions, potential splicing sites, and noncoding regions.
5) Perform Fisher's exact test on the wt/mutant counts for each variant for the mutant vs phenotypic wt libaries.
6) Perform Benjamini-Hochberg (BH) p-value adjustment for all filtered variants.
7) Manually determine the amino acid change from the GRCz11 annotation file for all coding variants and list the alternative reading frames and their potential amino acid changes. - This was originally implemented when GRCz10 was the most recent genome build for danio rerio and VEP was not available for predicting variant effects. Alternative reading frames were also calculated due to the poor quality of the zebrafish genome and annotations. It has been retained through GRCz11 due to gaps in VEP's annotation quality. I still recommend using VEP first to check variant effects on amino acids and the effect of variants.
8) Use VEP to classify identified coding variants amino acid modification and determine SIFT score for deleteriousness or tolerance of amino acid changes.
9) Use the adjusted p-values from 6 to build genome-wide and chromosomal manhattan plots. This borrows from an idea from GWAS.  Variants occuring near the causal variant are often retained in incrosses due to DNA crossover (recombination) during meiosis.  For forward genetic screens using this experimental strategy, a "region of homozygosity" within where the variant is located is usually observed.
10) Package analysis as well as QC parameters such as variant depths, variant quality, and distribution of variants across the genome.

## Usage

zvp is distributed as a singularity image. This can be downloaded via:
```
wget http://regmedsrv1.wustl.edu/Public_SPACE/pgontarz/Public_html/Zebrafish_variant_calling_and_prioritization/zvp.simg
```
Instructions to install singularity can be found here: https://sylabs.io/guides/3.0/user-guide/installation.html.  The singularity image is somewhat large but contains all necessary reference, customized annotation, and customized ensembl VEP database files to run locally. zvp is for use with GRCz11 as assemblies GRCz10 and zv9 are not supported by VEP.

zvp takes as input two vcf files - one for the variants called from the WGS or WES for the mutant and one from the sibling. zvp is compatable with either vcf files generated by GATK or bcftools. Each records allele counts in a different format. zvp automatically determines whether a vcf file comes from GATK or bcftools. In my experience, GATK and bcftools perform relatively similiarly for zebrafish WGS/WES. However bcftools is several times faster than GATK. If you start from raw reads files (.fastq.gz), you can prepare vcf files compatable with zfp via alignment with bwa followed by variant calling with bcftools as follows:
```
bwa mem -t <threads> /path/to/GRCz11.fa mutant_reads_1.fastq.gz mutant_reads_2.fastq.gz | samtools sort -@ <threads> - > mutant_sorted_aligned.bam
bcftools mpileup -d 100 -q 20 -Q 20 --threads <threads> -Ou -f /path/to/GRCz11.fa mutant_sorted_aligned.bam | bcftools call -Ou -mv | bcftools filter -s LowQual -e '%QUAL<20 || DP>100' > mutant_variants.vcf
```
Repeat the previous two steps for sibling data. Note that the bcftools options "-d" and "DP" specify depth.  For WGS, values of 100 have been tested to work well for removing extremely high depth variants from repeat or blacklist elements of the genome.  For WES, a higher value for depth, such as 500 is warranted as depths are generally much higher in WES than WGS.
Move or copy the vcf files into the your working directory. Your copy of zvp can be located anywhere. zvp can now be run as follows:
```
singularity run -B :./:/process /path/to/zvp.simg mutant_variants.vcf sibling_variants.vcf output_directory
```
As a note: what this command does is run the singularity container zvp.simg. -B instructs singularity to create a binding such that output can be written to outside the image.

## Output

```
output_directory
|--QC
|  |--Mutant_Depths.txt
|  |--Mutant_Qualities.txt
|  |--Mutant_vcf_Depths.txt
|  |--Mutant_vcf_Qualities.pdf
|  |--Sibling_Depths.txt
|  |--Sibling_Qualities.txt
|  |--Sibling_vcf_Depths.txt
|  |--Sibling_vcf_Qualities.pdf
|  |--VCF_hits_plot.pdf
|--Manhattan_plots
|  |--Allele_freq_Manhattan_plot.pdf
|  |--P-val_Manhattan_plot.pdf
|  |--Chromosomal_plots
|     |--Chr1_Manhattan_Plot.pdf
|     |--.....
|     |--Chr25_Manhattan_Plot.pdf
|     |--ChrMT_Manhattan_Plot.pdf
|--SNP_reports
|  |--CD_SNP_Report.xlsx
|  |--Noncoding_Region_SNP_Report.xlsx
|  |--Splicing_Region_SNP_Report.xlsx
|--Indel_reports
|  |--CD_indel_Report.xlsx
|  |--Noncoding_Region_indel_Report.xlsx
|  |--Splicing_Region_indel_Report.xlsx
|--SNP_visualization
   |--Mutant_SNPs.bg.gz
   |--Mutant_SNPs.bg.gz.tbi


```

