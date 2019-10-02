# CSC-FIMM GATK4 Germline SNPs and Indels
This repository stores gatk4 Germline short variant discovery (SNPs + Indels) workflows and testing results.   
It mainly contains: two WDL scrpits and their matched json files: haplotypecaller-gvcf-gatk4 and joint-discovery-gatk4-local. Both are according to the GATK Best Practices (June 2016).   
This document offering the step-by-step introduction about GATK4 Germline short variant discovery with GATK4 Docker images (Tested with GATK4.0.6.0) and Cromwell (Tested with v33) on CSC cPouta VM.  
In this CSC-FIMM GATK4 Germline SNPs and Indels Github Page, we offers workflow scripts which suitable for running on cPouta VM and testing results. Tested with Sample NA12878, NA12891 and NA12892, separately on cPouta VM, 24 cores, 117.2GB RAM, with Swap 99 GB.   
## **haplotypecaller-gvcf-gatk4:**
The haplotypecaller-gvcf-gatk4 workflow runs HaplotypeCaller from GATK4 in GVCF mode on a single sample according to the GATK Best Practices (June 2016), scattered across intervals.  
**Requirements/expectations:**  
  * One analysis-ready BAM file for a single sample (as identified in RG:SM)
  * Set of variant calling intervals lists for the scatter, provided in a file

**Outputs:**
  * One GVCF file and its index  

## **joint-discovery-gatk4-local:**
The second WDL implements the joint discovery and VQSR filtering portion of the GATK Best Practices (June 2016) for germline SNP and Indel discovery in human whole-genome sequencing (WGS) and exome sequencing data.   
**Requirements/expectations:**
  * One or more GVCFs produced by HaplotypeCaller in GVCF mode
  * Bare minimum 1 WGS sample or 30 Exome samples. Gene panels are not supported  

**Outputs:**
  * A VCF file and its index, filtered using variant quality score recalibration (VQSR) with genotypes for all samples present in the input VCF. All sites that are present in the input VCF are retained; filtered sites are annotated as such in the FILTER field.   

## **Software version requirements (see recommended dockers in inputs JSON):**
  * GATK 4 or later
  * Samtools (see gotc docker)
  * Python 2.7
  * Cromwell version support:
    * Successfully tested on v33 on cPouta VM
    * Does not work on versions < v23 due to output syntax  

## **Tool Prerequisites:**
  * virtul machine (VM) on CSC cPouta
  * docker-ce
  * Git

## **IMPORTANT NOTE:**
  * VQSR wiring. The SNP and INDEL models are built in parallel, but then the corresponding recalibrations are applied in series. Because the INDEL model is generally ready first (because there are fewer indels than SNPs) we set INDEL recalibration to be applied first to the input VCF, while the SNP model is still being built. By the time the SNP model is available, the indel-recalibrated file is available to serve as input to apply the SNP recalibration. If we did it the other way around, we would have to wait until the SNP recal file was available despite the INDEL recal file being there already, then apply SNP recalibration, then apply INDEL recalibration. This would lead to a longer wall clock time for complete workflow execution. Wiring the INDEL recalibration to be applied first solves the problem.
  * The current version of the posted "Generic germline short variant joint genotyping" is derived from the Broad production version of the workflow, which was adapted for large WGS callsets of up to 20K samples. We believe the results of this workflow run on a single WGS sample are equally accurate, but there may be some shortcomings when the workflow is modified and run on small cohorts. Specifically, modifying the SNP ApplyRecalibration step for higher specificity may not be effective. The user can verify if this is an issue by consulting the gathered SNP tranches file. If the listed truthSensitivity in the rightmost column is not well matched to the targetTruthSensitivity in the leftmost column, then requesting that targetTruthSensitivity from ApplyVQSR will not use an accurate filtering threshold. This workflow has not been tested on exomes. The dynamic scatter interval creating was optimized for genomes. The scattered SNP VariantRecalibration may fail because of two few "bad" variants to build the negative model. Also, apologies that the logging for SNP recalibration is overly verbose.
  * For the joint calling pipeline, because intervals-hg38.even.handcurated.20k.intervals is larger than 128000 Bytes, the default Dsystem.input-read-limits.lines is not enough.   
  ```bash
  Add `-Dsystem.input-read-limits.lines=500000` in your command line or add it in your config file. 
  ```
  Thus, example running command line as (more detailed info can be checked in TestingResult):
  ```bash
  sudo java -Dsystem.input-read-limits.lines=500000 -jar /media/volume/cromwell-33.1.jar run /media/volume/g-snps-indels/joint-discovery-gatk4-local.wdl -i /media/volume/g-snps-indels/joint-discovery-gatk4-local.hg38.wgs.inputs.json
  ```
  Here is the [Cromwell Reference Config File](https://github.com/broadinstitute/cromwell/blob/0ad9cb61f90276e3576f6ba135cab65c02f09a9e/core/src/main/resources/reference.conf#L157) which contains all the default _override_ settings for cromwell. (Latest update 14 Aug 2018)
  * The provided JSON is meant to be a ready to use example JSON template of the workflow. It is the user’s responsibility to correctly set the reference and resource input variables using the [GATK Tool and Tutorial Documentations](https://software.broadinstitute.org/gatk/documentation/).
  * Relevant reference and resources bundles can be accessed in [Resource Bundle](https://software.broadinstitute.org/gatk/download/bundle).
  * For help running workflows on the CSC Pouta VM or locally please view the following [Running Instructions](https://github.com/github-fish/csc-fimm-workflow/tree/master/cf-seq-format-conversion).
  * Swap is necessary, I tested with 99GB which can be adjust to smaller according your analysis scale.
  * To ensure docker can mount file with hard link which can save tons of space for you, store inputs data and cromwell execution directory in the Same Filesystem. Run the pipeline with sudo right. Unless, you will find that the intermediate files generate by docker with root right, then hard link will be invalid.
(Another possible [solve method](https://gatkforums.broadinstitute.org/wdl/discussion/9477/localization-via-hard-link-has-failed)
