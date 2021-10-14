---
title: ParaBricks-WGS
author: Ghanshyam Chandra, Chirag Jain
date: 2021-10-14 20:00:00 +0530
tags: [WGS,Sequence Alignment]
math: true
image: /htb/ParaBricks-WGS/DNA.png
---
Image source: Design Cells/iStock/Getty Images
### **Whole Genome Sequecing(WGS) of Human Genome with 50X coverage on Nvidia Ampere 100 GPU's**
Nvidia Ampere 100 GPU's exploits wide SIMD architecture to accelerate worlds most demanding HPC and AI workloads, and one of the key apllication in HPC is Whole Genome Sequencing of Human Genome. This blog aims to benchmark runtime comparison of WGS of Human Genome with 50X coverage with multiple GPU's to help understand how quickly we can sequence Human Genome with one of the fastest compute architecture in the world.
## PARABRICKS DNA*(fq2bam) pipeline

### Dataset:
1. **Refrence Data:** [HG38](https://github.com/broadinstitute/gatk/blob/master/src/test/resources/large/Homo_sapiens_assembly38.fasta.gz?raw=true)
2. **Sequencing Data:** [ERR194147](https://www.ebi.ac.uk/ena/browser/view/ERR194147?show=reads)

### Results:

#### Table
> Execution Time

| Number of GPU's     | Execution Time(s) (Alignment) | Execution Time(s) (WGS) |
| ----------- | ----------- | ----------- |
| 1      | 9657      |             11053      |
| 2   | 4924         |             5903       |
| 4   | 2720         |             3389       |
| 8   | 1695         |             2204       |

>                     Table 1: Execution Time

#### Plot

> Execution Time

![Execution Time](/htb/ParaBricks-WGS/BWA_Exec.png)
>                     Figure 1: Execution Time for Sequence Alignment (BWA).

> Speed Up

![Speed Up](/htb/ParaBricks-WGS/BWA_SpeedUp.png)

>                     Figure 2: SpeedUp as compared with single A100 GPU 

### Dataset:
1. **Refrence Data:** [GRCh38.p12](https://www.ncbi.nlm.nih.gov/assembly/GCF_000001405.38/)
2. **Sequencing Data:** SRR534301


## PARABRICKS RNA(rna_fq2bam) pipeline
### Datasets used


### Results
#### Table
> Execution Time

| Number of GPU's     | Execution Time(s) (Alignment) | Execution Time(s) (WGS) |
| ----------- | ----------- | ----------- |
| 1      | 1488      |             1529     |
| 2   | 987         |             1037       |
| 4   | 498         |             549       |
| 8   | 453         |             493       |

>                     Table 2: Execution Time

#### Plot

> Execution Time

![Execution Time](/htb/ParaBricks-WGS/STAR_Exec.png)

>                     Figure 3: Execution Time for Sequence Alignment (STAR).

> Speed Up
![Speed Up](/htb/ParaBricks-WGS/STAR_SpeedUp.png)

>                     Figure 4: SpeedUp as compared with single A100 GPU 

### Discussion

1. **Weak Scaling:**
2. **Strong Scaling:**

## Experimental Configurations

1. **Compute Cluster:** PARAM-SIDDHI AI
2. **Compute Node:** Single Compute Node \
 GPU: 8X A100-SXM4-40GB \
 CPU: 8X AMD EPYC 7742 64-Core Processor \
 RAM: 1 TB \
Storage: 8 PB (lustre fs)
3. **PARABRICKS:** Nvidia Clara ParaBricks: 3.6.0-1

