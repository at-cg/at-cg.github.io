---
title: ParaBricks-WGS
author: Ghanshyam Chandra, Chirag Jain
date: 2021-10-15 02:00:00 +0530
tags: [WGS,Sequence Alignment]
math: true
image: /assets/img/blogs/ParaBricks-WGS/DNA.png
---
<head>
  <link
    href="https://fonts.googleapis.com/css?family=Montserrat" rel="stylesheet"/>
  <link rel="stylesheet" href="../assets/css/main.css" />
</head>

Image source: Design Cells/iStock/Getty Images
### **Benchmarking human whole genome and RNA sequencing using Nvidia Ampere A100 GPUs**
Nvidia Ampere A100 GPUs exploit wide SIMT architecture to accelerate world's most demanding HPC and AI workloads, and one of the key application in HPC is sequencing of human genome and transcriptome. This blog aims to measure runtime required for genome and transcriptome sequencing using multiple GPUs to help understand how quickly sequence analysis can be done with one of the fastest compute architecture in the world.
## PARABRICKS DNA FQ2BAM pipeline

### Dataset:
1. **Reference genome:** [GRCh38](https://github.com/broadinstitute/gatk/blob/master/src/test/resources/large/Homo_sapiens_assembly38.fasta.gz?raw=true)
2. **Whole-genome sequencing run:** [ERR194147](https://www.ebi.ac.uk/ena/browser/view/ERR194147?show=reads) (Illumina HiSeq 2000 paired end sequencing, ~50x coverage)

### Results:

#### Table
> Execution Time

| Number of GPU's     | Execution Time(s) (alignment-phase) | Execution Time(s) (Overall) |
| ----------- | ----------- | ----------- |
| 1      | 9657      |             11053      |
| 2   | 4924         |             5903       |
| 4   | 2720         |             3389       |
| 8   | 1695         |             2204       |

>                     Table 1: Execution Time

#### Plot

> Execution Time

![Execution Time](/assets/img/blogs/ParaBricks-WGS/BWA_Exec.png)
>                     Figure 1: Execution time for sequence alignment (BWA).

> Speed Up

![Speed Up](/assets/img/blogs/ParaBricks-WGS/BWA_SpeedUp.png)

>                     Figure 2: Speedup relative to single A100 GPU 

**Scaling:** From Figure 2, we can clearly observe that with increase in number of GPUs, we are getting almost linear speedup, hence we can conclude that PARABRICKS DNA pipeline is strong scalable upto 8 GPUs.

## PARABRICKS RNA FQ2BAM pipeline

### Dataset:
1. **Reference genome:** [GRCh38](https://www.ncbi.nlm.nih.gov/assembly/GCF_000001405.38/)
2. **RNA-seq sample:** [SRR534301](https://www.ncbi.nlm.nih.gov/sra/?term=SRR534301) (Illumina HiSeq 2000 paired end sequencing)


### Results
#### Table
> Execution Time

| Number of GPU's     | Execution Time(s) (alignment-phase) | Execution Time(s) (Overall) |
| ----------- | ----------- | ----------- |
| 1      | 1488      |             1529     |
| 2   | 987         |             1037       |
| 4   | 498         |             549       |
| 8   | 453         |             493       |

>                     Table 2: Execution Time

#### Plot

> Execution Time

![Execution Time](/assets/img/blogs/ParaBricks-WGS/STAR_Exec.png)

>                     Figure 3: Execution time for sequence alignment using STAR algorithm.

> Speed Up
![Speed Up](/assets/img/blogs/ParaBricks-WGS/STAR_SpeedUp.png)

>                     Figure 4: Speedup relative to single A100 GPU 

### Discussion

**Scaling:** In Figure 4, we observe that while keeping data set fixed and with increase in number of GPUs, we are getting almost linear scaling up to 4 GPUs and then speedup reduces. Hence, PARABRICKS RNA pipeline is showing good strong scaling up to 4 GPUs.

## Experimental Configurations

1. **Compute Cluster:** PARAM-SIDDHI AI
2. **Compute Node:** Single compute node \
 GPU: 8X A100-SXM4-40GB \
 CPU: Dual AMD EPYC 7742 64-Core Processor \
 RAM: 1 TB \
Storage: 8 PB (lustre fs)
3. **PARABRICKS:** Nvidia Clara ParaBricks: 3.6.0-1

