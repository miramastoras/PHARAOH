# PHARAOH - **PHA**sing **R**eads in **A**reas **O**f **H**omozygosity


### Overview

PHARAOH is a pipeline used for correcting the phasing of HiFi reads aligned to a diploid assembly. PHARAOH uses ONT UL reads to phase HiFi reads in regions where the diploid assembly is falsely homozygous for stretches longer than the length of the HiFi reads. For the rest of the genome, PHARAOH uses secondary alignments to re-assign reads to the correct haplotype, as implemented in [secphase](https://github.com/mobinasri/secphase). PHARAOH was developed to provide highly accurate read phasing to the DeepPolisher model for polishing the HPRC samples.

![pharaoh](images/PHARAOH.png)
