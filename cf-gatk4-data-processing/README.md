# CSC-FIMM GATK4 Data Processing
This repository stores gatk4 data pre-processing workflows and testing results.   
This document offering the step-by-step introduction about gatk4 data pre-processing with GATK4 Docker images (Tested with GATK4.0.4.0) and Cromwell (Tested with v33) on CSC cPouta VM.  
The main steps´description can be checked from [GATK Best Practice](https://software.broadinstitute.org/gatk/best-practices/workflow?id=11165) and its matched [Github Page](https://github.com/gatk-workflows/gatk4-data-processing).  
In this CSC-FIMM GATK4 Data Processing Github Page, we offers workflow scripts which suitable for running on cPouta VM and testing results. Tested with Sample NA12878, NA12891 and NA12892, separately on cPouta VM, 24 cores, 117.2GB RAM, with Swap 99 GB.   
## **processing-for-variant-discovery-gatk4:**
The processing-for-variant-discovery-gatk4 WDL pipeline implements data pre-processing according to the GATK Best Practices.   
**Requirements/expectations:**  
  * Pair-end sequencing data in unmapped BAM (uBAM) format
  * One or more read groups, one per uBAM file, all belonging to a single sample (SM)
  * Input uBAM files must additionally comply with the following requirements:
    * filenames all have the same suffix (we use ".unmapped.bam")
    * files must pass validation by ValidateSamFile
    * reads are provided in query-sorted order
    * all reads must have an RG tag
  * Reference index files must be in the same directory as source (e.g. reference.fasta.fai in the same directory as reference.fasta)

**Outputs:**
  * A clean BAM file and its index, suitable for variant discovery analyses.  

**Software version requirements (see recommended dockers in inputs JSON):**
  * GATK 4 or later
  * Picard (see gotc docker)
  * Samtools (see gotc docker)
  * Python 2.7
  * Cromwell version support:
    * Successfully tested on v33 on cPouta VM
    * Does not work on versions < v23 due to output syntax  

**Tool Prerequisites:**
  * virtul machine (VM) on CSC cPouta
  * docker-ce
  * Git  

**Important Note:**
  * The provided JSON is meant to be a ready to use example JSON template of the workflow. It is the user’s responsibility to correctly set the reference and resource input variables using the [GATK Tool and Tutorial Documentations](https://software.broadinstitute.org/gatk/documentation/).
  * Relevant reference and resources bundles can be accessed in [Resource Bundle](https://software.broadinstitute.org/gatk/download/bundle).
  * For help running workflows on the CSC Pouta VM or locally please view the following [Running Instructions](https://github.com/github-fish/csc-fimm-workflow/tree/master/cf-seq-format-conversion).  
  * Swap is necessary, I tested with 99GB which can be adjust to smaller according your analysis scale.
  * To ensure docker can mount file with hard link which can save tons of space for you, store inputs data and cromwell execution directory in the Same Filesystem. Run the pipeline with sudo right. Unless, you will find that the intermediate files generate by docker with root right, then hard link will be invalid.
(Another possible [solve method](https://gatkforums.broadinstitute.org/wdl/discussion/9477/localization-via-hard-link-has-failed)
