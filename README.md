# HIF-1-driven-stemness-in-CSC-murine
HIF-1α-Driven Stemness Maintenance in Cancer Stem Cells (PRJNA1442098) 

Keywords:
Hypoxic response, Stemness response, HIF-1α-Driven Stemness, CSC, TCGA, Survivability studies

I started this project to learn more about the use of marker-based enrichment strategy (like FACS for CD133+ vs CD133- pools) for building a high-sensitivity baseline of stemness transcription factors. 
Complete project was planned in several phases<br>
> (1) Upstream processing -> setting up the DEseq2 pipeline to obtain signatures specific to this process<br>
> (2) Downstream processing -> discover the best ways to deconvolve public TCGA data using signatures<br>
<br>
Data used for the analysis was taken from [1]. In this project, there were five libraries:
 - HIF-1α overexpression (OE)
 - shControl (WT)
 - HIF-1α knockdown (KD)
 - dynamic suspension culture (WT)
 - conventional three-dimensional culture (WT).

# 1. Complete processing WF

Complete processing WF is shown on Figure 1.

![Complete WF](Images/Complete_flow_uml.png)

**Figure 1: Complete processing WF**

# 2. Phase I - RNA-seq profiling

Data used in this study is RNA-seq, so appropriate WF was created. The processing steps were as follows (see Figure 1):
 - Quality control (FastQC/MultiQC)
 - Adapter trimming (fastp)
 - Mapping to reference (STAR)
 - Creation of count matrices (featureCount)
 - Differential expression analysis (DESeq2).

Based on libraries obtained in this project, following samples were used in the DESeq2 analysis:<br>
 - WT cell populations (created using different methods)
 - OE cell populations
 - KD cell populations.

DESeq2 analysis performed was successful despite limited number of replicate, which is sometimes common (e.g. when using public data or difficult enrichment protocols).

**Explanation**<br>
Because the DESeq2 was setup with 3 WT samples, their variance was used to estimate a global dispersion.<br>
This allowed the model to function, although the p-value assignments for the KD and OE groups were not accurate.<br>

After performing Variance stabilizing Transformation (VST) to account for heteroskedastic nature of RNA-seq data, the PCA plot revealed that the WT cell populations, that were created using different methods, are pulling away from each other and KD/OE cell cultures (see Figure 2):<br>

![PCA plot](Images/PCA_plot.png)

**Figure 2: PCA plot**

Because of that situation, it was important to identify which WT cell type is which in the plot. This was made using enhanced PCA plot (Figure 3).

![PCA plot](Images/PCA_plot_enh.png)

**Figure 3: PCA plot - enhanced w/Metadata**

In this study, there are following WT samples:<br>
 - WT1 = conventional three-dimensional culture (SRR37746879)<br>
 - WT2 = dynamic suspension culture (SRR37746878)<br>
 - WT3 = shControl (SRR37746876).<br>
<br>

## Breakdown of PCA with Metadata

Following conclussions can be made based on this PCA plot: <br>
 - WT1 (3D Static suspension) & WT2 (3D Dynamic suspension) --> Transition from static to dynamic suspension had a smaller impact on the transcriptome than the viral transduction/selection process
 - WT3 (shControl) --> WT3 subjected to viral transduction/selection process
 - KD & OE --> These are shifted along PC1 relative to the "shControl" baseline.

*Conclussion*<br>
Gene expression on KD cell culture should not be compared directly to WT1 or WT2, "shControl" (WT3) is the only valid baseline for the HIF-1α manipulation, as it accounts for the stress of transduction.<br>

This setup also implies that we are in fact dealing with two independent experiments: <br>
 > Experiment A --> How does physical stress in the tumor microenvironment (Dynamic Culture) affect the stemness (HIF-1α expression) compared to a standard static 3D culture (WT1, WT2)<br>
 > Experiment B --> How does changing the HIF-1α expression (upregulation OE using plasmids/viral vectors vs. KD using shRNA) affect the stemmness (OE vs KD vs WT3).<br>
<br>
*Conclusion*<br>
A standard DESeq2 workflow that calculates dispersion can not be used due to the lack of replicates.<br>
This means that the focus of analysis should move from Statistical Discovery to Pattern Matching with the final aim of building set of genes for finding a signature in the TCGA.<br>
<br>
## Calculating the fold change
<br>
Instead of looking for statistically significant differentially expressed genes, we should be looking for **consistently ranked** genes. The approach taken is:<br>
>  1.  Normalize the count matrix to TPM --> This accounts for gene length and sequencing depth<br>
>  2.  Calculate the delta (FC) --> Find genes where the "dosage" of HIF-1α perfectly matches the expression: KD < WT3 < OE.<br>
>  3.  The Intersection Filter:<br>
  >  -   Find the top variable genes for HIF-1α stemmness factor (OE vs KD vs WT3)
  >  -   Find the top variable genes related with Physical Stress (WT2 vs WT1)

The intersection of these two lists is **HIF Stemness Signature**<br>

Based on these findings, it is possible to use the standard way to validate experimental signatures on patient survival using the following steps:<br>
 - Compute a signature from experimental data (HIF Stemness Signature)<br>
 - Map mouse genes to human orthologs, annotate (Biomart)<br>
 - Test that signature in TCGA patient cohorts (TCGA-SKCM)<br>
 - Build a signature score per TCGA sample by applying Signature gene set to a client TCGA expression matrix (compute z-score as mean(log2(TPM+1))<br>
 - Match TCGA expression samples to clinical data<br>
 - Run Kaplan–Meier / Cox using the TCGA samples grouped by your signature (High vs Low) or using the signature score as continuous predictor.<br>
<br>
These procedures are described in the following sections.<br>
<br>
Heatmap of stemmness signature (see Figure 3) provides visualization of the signature strength and shows the consistency of expressed genes across the 5 samples.<br>

![Heat map](Images/Heatmap_HIF_Stemness_signature.png)

**Figure 3: Heatmap of HiF stemness signature (murine)**

## 2. Phase II - Translation and mapping

For Gene signature obtained in the previous step (Phase I) to be used in subsequent phases of analysis, the genes need to be first translated using human --> mouse ortholog mapping and annotated. <br>
The identified Gene expression profile in a murine model leads to HIF-1α activation. In an independent human melanoma cohort (TCGA-SKCM), this signature could function as statistically significant prognostic marker. <br>
The prognostic signature obtained in this way can then be used to test survivability.<br> 

After conversion, some of the genes are lost due to the fact that they are missing human orthologs. <br>
The table of significant markers for HIF-1α signature are shown in the Table 1. <br>

![Significant markers](Images/final_list.png)

**Table 1: Significant markers - human orthologs**

Final list contains some genes that are relevant for this process (e.g. BNIP3, CD274 (PD-L1), DNAJB6, TP53INP1 etc.).<br>
This proves that the assay was able to capture HIF-1α related signaling, even in the absence of biological replicates and statistically significant genes.<br>

## 3. Phase III - Human cohort scoring

For testing the gene signature agains TCGA database, the TCGA-SKCM (Skin Cutaneous Melanoma cohort) was used.<br> 
The data from *tpm-unstrand* assay was used to calculate the z-score for each gene in signature across all patients.<br>

## 4. Phase IV - Clinical validation

In order to perform survival analysis, the scores obtained were attached to clinical data and fitted to survival model.<br> 
To perform survival plot, median split on all samples was used.<br> 

The survival plot is shown on the Figure 4.

![Survivability](Images/Survival_validation_of_Mouse_HIF-stem_signature.png)

**Figure 4: Survival plot**

**Conclussion**
Mouse-derived gene signature was succesfuly evaluated against a large human cohort.<br>
Statistical Significance (p = 0.033) obtained in the Kaplan-Meier plot is below the standard 0.05 threshold.<br>

*Clinical Relevance*<br>
The "High Signature" (Yellow) group consistently maintains a higher survival probability than the "Low Signature" (Blue) group over the entire study period.<br>
This suggests that this HIF-1α signature, obtained using a murine-derived physiology, defines a less-aggressive phenotypic subgroup in human melanoma.<br>

References:
  [1] Dynamic Suspension Culture System Reveals HIF-1α-Driven Stemness Maintenance via Dual Suppression of the p53 Pathway in Cancer Stem Cells (house mouse), NCBI PRJNA1442098 
