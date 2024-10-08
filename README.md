# Metagenomics Binning Workshop

Welcome to the Metagenomics Binning Workshop from Computational Metagenomics (BioSB Advanced course)! This repository contains materials, scripts, and instructions for learning how to bin metagenomic data into high-quality genome bins. In this workshop, we will explore the various tools and methods used for metagenomic binning and assembly of microbial genomes from complex environments.

## Table of Contents
- [Workshop Overview](#workshop-overview)
- [Introduction](#introduction)
- [Datasets](#datasets)
- [Quality Control](#quality-control )
- [Read-based approches](#read-based-approches)
- [Assembly](#assembly)
- [Binning](#binning)
- [Investigation](#Investigation)
## Workshop Overview
Metagenomics binning is a key step in reconstructing genomes from microbial communities, allowing researchers to study unculturable microorganisms. In this workshop, you will:

- Understand the principles of metagenomic binning.
- Learn how to quality control, filter and pre-process raw megaenomics reads.
- Assembly your reads to contigs with popular tools, such as `MEGAHIT`.
- Use popular tools such for binning such as `MetaBAT`.
- Evaluate the quality of asseblies and genome bins with tools like `CheckM` and `QUAST`.

The workshop includes hands-on exercises with real metagenomic datasets and step-by-step guidance on a simple binning apporch.

## Introduction
Dataset explanation:

- In this practical session, you will process raw sequencing reads to achieve taxonomic characterization of metagenome-assembled genomes (MAGs).
- For this experiment, we have collected two samples from a toxic biofilm in a Romanian cave that emits greenhouse gases, along with one methane-enriched sample from the lab.
- Your task is to identify the community members capable of methane consumption and the associated gene clusters responsible for this process.

## Datasets
For this workshop, we will use publicly available metagenomics datasets with the following command:
```bash
wget -nc ftp://ftp.sra.ebi.ac.uk/vol1/run/ERR100/ERR10036470/*
```

The datasets from their respective sources or from the provided links in the `datasets/` folder.

## Quality Control 
> [!Learning Objective]
> Useful information that users should know, even when skimming content.
<div style="background-color: #f0f8ff; border: 1px solid #dcdcdc; padding: 10px; border-radius: 5px;">
  
Learning Objective
  
- Understand the basic concepts of X
- Learn the principles of Y
- Apply the knowledge of Z to solve real-world problems
  
</div>
> **Important: Learning objectives**
> - Understanding the FastQ format
> - Interpret FastQC reports
> - Create high quality reads by trimming and filtering with FastP and BBduk

## FastQ format

First let's take a look at the data

    zcat /data/reads/sample_0.fq.gz | head


??? done "The output looks like this"
        @gi|1184849861|gb|KY629563.1|-34525/1
        CTGATCAACGTGGGAATGAACATGGAGCAGAGCGGCCAGTGTCTCAACCTGTGCTCCGCCTTCATGCACTCTCGCGGCGCCTTCAAGCCCGGCGACATCGACGTGGAAATTTTCACCATGCCCACCAAGTTCGGGCGCGTCAGCACCACG
        +
        DDDGGEG*DIIGH?KKHJGHGKKKJKKJJ9JDK=JKFKKGJHIKHEIJJJKKGIHKGDE1DKA>KEGBEEEBGI?BBEGHEEEFEEE9ED;FCB3BEEDCBEE5EEC'CA$E9EEEAEEEFEE?FFCEEE;$EE)DBCDEEDDECAAE'$
        @gi|1184849861|gb|KY629563.1|-34523/1
        GCCTTTGTCGGGTAAGGGGTGTGGCCCTCCTCCCGACAAGGCGGGCCACGGTTCGCCAGCGAACTAGGTCGGGGAGCTGGGAAGGAGCCGGAATCGGGTGGCCCCAATTTCGGGGAGAGGTTTGGGCGTCAGCCGCCCGGAAGCTCGTCG
        +
        DDDE2GGGIIIIIKKJKHJKEDJ=AIHKKHKJKKKKJHBIE$KJJJCEKJB=JH@JKEHKGEEEEDAJECDGC?EIFEBEDDF6FGEEDEE$FBDCEEA@EE$4EE$E??CDE?ED;CCDE1DEBE;ECC$?$$AEDA;EDD@$EEDE=F
        @gi|1184849861|gb|KY629563.1|-34521/1
        AGACCGAGCCCTTCCTTATAGTGGATTTCTCCGGTTCCGTCAACCAAATTTGCAGTAATGCGGGAGCGACGCTTCCACGAAATGGTCACTGTCCCGTCCACCTCGGAGAGCTTTACGCTTTCCGGTGTGTAGGGCTTGAGATCGATCGAT

There is a pattern that is repeated every 4 lines. This is the [FASTQ format](https://en.wikipedia.org/wiki/FASTQ_format). It follows this structure:

|Line|Description|
|----|-----------|
|1|Always begins with '@' and then information about the read|
|2|The actual DNA sequence|
|3|Always begins with a '+' and sometimes the same info in line 1|
|4|Has a string of characters which represent the quality scores; must have same number of characters as line 2|

The fourth line shows the quality of the read. To make the sequence and the qaility align well, the numerical score is converted into a code where each individual character represents the numerical quality of an individual nucleotide, following this scheme:

!!! done "FASTQ quality encoding"
            Quality encoding: !"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJ
                            |         |         |         |         |
            Quality score:    01........11........21........31........41

NextSeq and NovaSeq data often contains poly-g tails. Let's check if this also is the case for our simulated input

    $ zcat /data/reads/sample_0.fq.gz | grep -E "GGGGGGGGGGGGGGGGGG$"

This search does not retun any hits. But often the end of the sequences contain stretches of poly-g. Below is an example how poly-g tails look like in Novaseq data.

??? done "Sequence data with poly-g tails"
        GGCCAGGACCACGCGGTGGAGCAGCGCGCCGCGGCGCCGGGCGCGCTTGACGACCAGTCTCTTATAAACATATCCCAGCCCACGAGACCACCAGTTGCATCTCGTATTCCGTCTTATGCTTGTATATTGGGGGGGGGGGGGGGGGGGGGGG
        GAGTCGATCGAGGAGATGAAGCACGCGGAGAAGGTCATTCACCGCATCCTCTACTTCGATGCTGTCTCTTATACACATCCCGAGCCCACGAGACCACCTGTTGCATCTCGTATGCCGTCTTCTGCTTGAAAAGGGGGGGGGGGGGGGGGGG
        GGCCCGTGCAGTTCGAGATCATCTCCGAGCCCACGAGACGACCGGGTGCTTGGCGGGAGCGGCGGGGTGGTTTTATCTTCGTGGTGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGG
        GGGCGGCAACGCCGACACCTACCAGCTACAGCAGACGGTGCGCCGCACCATCGGCCGCTGGGTCGGCGGCCGGCTGCGCCGCCGTCCCAAGATAATCCCGGTGGTGGGGGGGGGTGGTTGGGGGGGTTGTGGGGGGGGGGGGGGGGGGGGG
        GGTAGGGCCGCTCGAGAAGCTCGCACAGCATGCGGCCGAATTCGCGGTACATGCATACGTTGACGTCGGCGGGGGGGGGGGGGGGGGGAGAGCACCGGGGGGCGCACAGGGCGGCCGGGTGGGGGTCGTGGTGGGGGGGGGGGGGGGGGGG
        GGGGAGGAGGGAGACGCGCCCGCGGGCGTGCCCGCCGCCGCGGCGATGCCCTTGAAGGCCGCTGGTGTGTCCTTCAGCGCCTGCGCCGCGAGTTCGCCGAACTGCTGCGTCAACGCGCCCCACCACTGCTGCGGGGGGGGGGGGGGGGGGG
        GTACCCGAGCCGCTCCGCGTGCCGCCGGACCTCCGACGCGTGAGGCCCGGAGCCGGTCGCTGACCAGAGGGCCAGGCCGAAGCGATGGGGAGTGGTCGCGACGAACCCGCAGCGCTGTCCGGCGGCGGGGGGGGGGGGGGGGGGGGGGGGG
        GAAGGTGTGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGTGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGCGGGGGGGGGGGGGGGCGGGGGCGGGGGGGGGGGGGGGGGGGGGGGGGGGG
        GCGCCACGCCGAGCACCGACGGCATCATCGGCACCCACACGCCGTGTGTGAACCTGTCTCTTATACACATCTCCGAGCCACGAGACCACCTGTTGCATCTCGTATGCCGTCTTCTGCTTGAAAATGGGGGGGGGGGGGGGGGGGGGGGGGG
        CTCTTCATCCGTTCCGGCGCCTGCATCCATTCCCGCGGCGCCGGTTGGCGGGGGGGGGGGGGGGGGGGGGGGGGGTGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGG

## FastQC reports

Let's check the quality of the data with FastQC before quality filtering:

    mkdir fastqc_untrimmed_reads
    fastqc --threads 8 /data/reads/sample_0.fq.gz -o fastqc_untrimmed_reads/

!!! question Exercise
    Open the FastqQC report `fastqc_untrimmed_reads//sample_0_fastqc.html`

## Quality trimming and filtering

Because NovaSeq and NextSeq from Illumina contain often poly-g tails, it is good to use a trimming tool that can detect poly-g tails. Fastp can do that. We keep the defaults like they are, but specify we have interleaved input data. In case you have R1 and R2 file for each sample, you need to enable adapter detection with the `--detect_adapter_for_pe` flag. Execute fastp with this command:

    fastp -i /data/reads/sample_0.fq.gz \
          --stdout \
          --interleaved_in \
          -q 25 \
          --cut_front \
          --cut_tail \
          --cut_mean_quality 25 \
          -l 51 \
          --thread 16 \
          --trim_poly_g > sample_0.trim.fastq

Now make a FastQC report again, to see the results of the quality filtering.

    mkdir fastqc_trimmed_reads
    fastqc --threads 8 sample_0.trim.fastq -o fastqc_trimmed_reads/

!!! question "Exercise" 
    - Are there any adapter sequences detected?
    - Take a look at the html report

### Alternative trimming with bbduk

Alternative trimming with bbduk. Compare poly-g tail filtering, adapter trimming.

    bbduk.sh in=/data/reads/sample_0.fq.gz  \
        out=sample_0.trim.bbduk.fastq.gz \
        interleaved=true \
        trimpolygright=1 \
        qtrim=w trimq=20 \
        minlength=51 \
        ref=/data/databases/nextera.fa.gz ktrim=r \
        stats=bbduk.stats \
        t=16

!!! question Exercise
    - Create also an FastQC report for the trimming with bbduk 
    - What are differences between filtering with fastp and bbduk?

## Remove PhiX sequences

    bbmap.sh ref=/data/databases/phix174_ill.ref.fa.gz \
             in=sample_0.trim.fastq \
             interleaved=true \
             outu=sample_0.nophix.fastq.gz \
             outm=sample_0.phix.fastq.gz \
             t=4


!!! question "Exercise" 
    - How many PhiX sequences are detected?
    - From which samples?
    - Can you confirm the sequences are PhiX?
    - The input data was simulated without adding any PhiX. How could there still PhiX sequences being detected?

## From one sample to many

!!! question "From one sample to many"

    Now do a QC for all samples. You can use a for loop for that. For example, fastp can be run like:

        for file in /data/reads/*.gz; do \
            sample=$(basename ${file} .fq.gz);
            fastp -i $file --stdout --interleaved_in -q 25 --cut_front --cut_tail --cut_mean_quality 25 -l 51 --thread 16 --trim_poly_g > $sample.trim.fastq;
            bbmap.sh ref=/data/databases/phix174_ill.ref.fa.gz in=$sample.trim.fastq interleaved=true outu=$sample.nophix.fastq.gz outm=$sample.phix.fastq.gz t=4;
        done

## Quickly check what is in this metagenome

### Sendsketch
    sendsketch.sh --in=sample_0.nophix.fastq.gz threads=4 address=refseq

### Sourmash
**Note**: The sourmash database is not included in the VM because it can't be downloaded at the moment. There is an [overview of all prepared databases](https://sourmash.readthedocs.io/en/latest/databases.html) The [51 kmer set of representative genomes](https://farm.cse.ucdavis.edu/~ctbrown/sourmash-db/gtdb-rs207/gtdb-rs207.genomic-reps.dna.k51.lca.json.gz) would be a good one to use when available

Now try it locally, using sourmash. First create a signature for a sigle sample.

    sourmash sketch dna -p scaled=10000,k=51 sample_0.nophix.fastq.gz -o sample_0.sig  

Find what is in this metagenome using the `gather` command:

    sourmash gather sample_0.sig /data/databases/gtdb-rs207.genomic-reps.dna.k51.lca.json.gz

### Kraken
Compare with kraken

First setup the database

    mkdir kraken_db
    tar zxvf /data/databases/k2_standard_08gb_20220926.tar.gz -C kraken_db

Now run kraken2

    kraken2 --db kraken_db/ --threads 16 --output sample_0.kraken --report sample_0.kraken.report --gzip-compressed --use-names sample_0.nophix.fastq.gz

!!! question 
    Take a look at the `sample_0.kraken.report`. What are the differences in classification compared to sendsketch and sourmash?
## Read-based approches
To install the necessary software, follow these steps:

1. Clone this repository:
   ```bash
   git clone https://github.com/your_username/metagenomics-binning-workshop.git
   cd metagenomics-binning-workshop
   ```



## Read-based approches


## Assembly


## Binning

## Investigation
