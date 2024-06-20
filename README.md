# PHARAOH - **PHA**sing **R**eads in **A**reas **O**f **H**omozygosity


### Overview

PHARAOH is a pipeline used for correcting the phasing of HiFi reads aligned to a diploid assembly. PHARAOH uses ONT UL reads to phase HiFi reads in regions where the diploid assembly is falsely homozygous for stretches longer than the length of the HiFi reads. For the rest of the genome, PHARAOH uses secondary alignments to re-assign reads to the correct haplotype, as implemented in [secphase](https://github.com/mobinasri/secphase). PHARAOH was developed to provide highly accurate read phasing to the [DeepPolisher model](https://github.com/google/deeppolisher) for polishing the HPRC samples.

![pharaoh](images/PHARAOH.png)

### Steps:
  1.  Align haplotype 1 assembly to haplotype 2 assembly, and [detect homozygous regions](https://github.com/mobinasri/secphase?tab=readme-ov-file#detecting-homozygous-regions).
  2. Align HiFi reads to the diploid assembly using minimap2 or winnowmap. Extract the reads in the homozygous regions identified by step 1, filtering out reads with too high a divergence from the assembly (de > 0.02).
  3. Align homozygous reads to each haplotype using minimap2 or winnowmap
  4. Call variants with [deepvariant](https://github.com/google/deepvariant), extracting just homozygous calls  
  5. Align ONT UL reads > 100kb to each haplotype separately. Using these reads, apply Margin or WhatsHap to phase the homozygous variants called by DeepVariant in Step 4.
  6. Run secphase in dual mode [as described here](https://github.com/mobinasri/secphase?tab=readme-ov-file#running-secphase-in-dual-mode). Secphase will use phased variants from step 5 to assign reads in homozygous regions to the correct haplotype. In non-homozygous regions, marker-mode of secphase will be applied.
  7. Final alignment is produced, with read phasing fixed in homozygous regions.


![pharaoh](images/PHARAOH_overview.png)

### Generating inputs to PHARAOH



### Running PHARAOH

Currently PHARAOH is implemented as a wdl workflow. A WDL file can be run locally using Cromwell, which is an open-source Workflow Management System for bioinformatics. The latest releases of Cromwell are available [here](https://github.com/broadinstitute/cromwell/releases) and the documentation is available [here](https://cromwell.readthedocs.io/en/stable/CommandLine/).

The wdl to use for PHARAOH is located under [wdl/workflows/PHARAOH.wdl](https://github.com/miramastoras/PHARAOH/blob/main/wdl/workflows/PHARAOH.wdl).

The command to use for running PHARAOH.wdl with cromwell is:
```
java -jar cromwell-85.jar run \
    wdl/workflows/PHARAOH.wdl \
    -i inputs.json \
    -m outputs.json
```

All required parameters must be supplied to inputs.json file. You may use womtool to generate an example inputs.json file, and pass in your files.
```
wget https://github.com/broadinstitute/cromwell/releases/download/85/womtool-85.jar
womtool-85.jar womtool-85.jar wdl/workflows/PHARAOH.wdl > inputs.json
```

Below are the reccommended parameters to review for your inputs.json file:
```
{
  "PHARAOH.allHifiToDiploidBam": "File",
  "PHARAOH.allHifiToDiploidBai": "File",
  "PHARAOH.allONTToHap1Bam": "File",
  "PHARAOH.allONTToHap1Bai": "File",
  "PHARAOH.allONTToHap2Bam": "File",
  "PHARAOH.allONTToHap2Bai": "File",
  "PHARAOH.Hap1Fasta": "File",
  "PHARAOH.Hap1FastaIndex": "File",
  "PHARAOH.Hap2Fasta": "File",
  "PHARAOH.Hap2FastaIndex": "File",
  "PHARAOH.PharaohAligner": "String (optional, default = \"minimap2\")",
  "PHARAOH.PharaohHiFiPreset": "String (optional, default = \"map-hifi\")",
  "PHARAOH.diploidFaGz": "File",
  "PHARAOH.pafAligner": "String (optional, default = \"minimap2\")",
  "PHARAOH.PharaohKmerSize": "String (optional, default = 19)",
  "PHARAOH.useMargin": "Boolean (optional, default = false)",
  "PHARAOH.sampleName": "String"
}
```

Detailed description of these parameters for PHARAOH.wdl:

| **Parameter** | **value** |
|---------------|-----------|
| PHARAOH.allHifiToDiploidBam| Location of bam file with all HiFi reads aligned to diploid assembly. Can be generated by [minimap2](https://github.com/lh3/minimap2) or [winnowmap](https://github.com/marbl/Winnowmap)|
|PHARAOH.allHifiToDiploidBai | .bai index file for allHifiToDiploidBam. Use [samtools index](https://www.htslib.org/doc/samtools-index.html) to generate |
|PHARAOH.allONTToHap1Bam |Location of bam file with all ONT reads aligned to Haplotype 1. Can be generated by [minimap2](https://github.com/lh3/minimap2) or [winnowmap](https://github.com/marbl/Winnowmap) |
|PHARAOH.allONTToHap1Bai| .bai index file for allONTToHap1Bam |
|PHARAOH.allONTToHap2Bam |Location of bam file with all ONT reads aligned to Haplotype 2. Can be generated by [minimap2](https://github.com/lh3/minimap2) or [winnowmap](https://github.com/marbl/Winnowmap)|
|PHARAOH.allONTToHap2Bai|.bai index file for allONTToHap2Bam |
|PHARAOH.Hap1Fasta| .fasta file containing haplotype 1|
|PHARAOH.Hap1FastaIndex| .fai index file for Hap1Fasta. Use [samtools faidx](https://www.htslib.org/doc/samtools-faidx.html) to generate |
|PHARAOH.Hap2Fasta|.fasta file containing haplotype 2|
|PHARAOH.Hap2FastaIndex|.fai index file for Hap2Fasta. Use [samtools faidx](https://www.htslib.org/doc/samtools-faidx.html) to generate |
|PHARAOH.PharaohAligner| Either "minimap2" or "winnowmap" |
|PHARAOH.PharaohHiFiPreset| For PharaohAligner="minimap2" use preset "map-hifi" and for PharaohAligner="winnowmap" use preset "map-pb"|
|PHARAOH.diploidFaGz| diploid assembly fasta file. Needs to be gzipped. |
|PHARAOH.pafAligner| Either "minimap2" or "winnowmap" |
|PHARAOH.PharaohKmerSize| For PharaohAligner="minimap2" use preset "map-hifi" and for PharaohAligner="winnowmap" use preset "map-pb"|
|PHARAOH.useMargin| Boolean. True means use Margin for phasing variants in homozygous regions with the UL alignments, False means use WhatsHap. WhatsHap is the default and currently reccommended.|
|PHARAOH.sampleName| name of sample being run|
