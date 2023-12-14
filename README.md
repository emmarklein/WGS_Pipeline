# Whole-Genome Sequencing (WGS) Pipeline

This repository includes everything you need to work with WGS data. Raw FASTQ files serve as input and undergo genome alignment to a reference. We will go through this process step by step using the T2T and hg38 references.

## SRA download and splitting
Let’s start with raw FASTQ files (.fastq). FASTQ files contain our sequencing information. We can download Sequence Read Archive (SRA) data from: https://www.ncbi.nlm.nih.gov/sra. This site stores raw sequencing data and alignment information. I will be using SRR8670768 as example data!

```
module load sratoolkit
prefetch --max-size 1000000000000 --force all -q SRR8670768
```

The output should be the complete .SRA file from the site above. Next, we need to split the SRA file because it includes paired-end reads. Let’s continue using sratoolkit! 

```
module load sratoolkit
fastq-dump --split-files --gzip -O /your/path/SRR8670768 /your/path/SRR8670768/SRR8670768.sra
```

## Decompressing fastq.gz

Both reads (SRR8670768_1.fastq.gz and SRR8670768_2.fastq.gz) need to be decompressed! We can use gunzip for this step.

```
gunzip -c /your/path/SRR8670768_1.fastq.gz > /your/path/SRR8670768_1.fastq
gunzip -c /your/path/SRR8670768_2.fastq.gz > /your/path/SRR8670768_2.fastq
```

Now, we finally have our unzipped raw fastq files!

## FastQC

FastQC can be used for quality control checks of the high throughput data and eliminated poor quality reads.

```
module load fastqc/0.12.1
fastqc -o /your/path/fastqc_output /your/path/SRR8670768_2.fastq
```

## Trimgalore!

Let's use trim_galore to trim the reads by removing adapter sequences. This command uses the zipped version of the fastq files, but you can also run this with unzipped fastq files!

```
module load trim_galore/0.6.7
module load cutadapt/4.4
trim_galore --paired -q 24 --fastqc -o /your/path/SRR8670768_trimmed_new /your/path/SRR8670768_1.fastq.gz /your/path/SRR8670768_2.fastq.gz
```

## Burrows-Wheeler Alignment (BWA)

The first step of using BWA is to make an index of the reference genome in fasta format. We can use bwa index!

```
module load bwa/0.7.17
bwa index -p bwa_hg_index /your/path/GRCh38.primary_assembly.genome.fa
bwa index -p bwa_t2t_index /your/path/GCF_009914755.1_T2T-CHM13v2.0_genomic.fna
```

Now, we can do alignment with bwa mem!

```
module load bwa-mem2/2.2.1
module load bwa-meth/0.2.4

#bwa for hg38
bwa mem -t 4 bwa_hg_index /your/path/SRR8670768_1_trimmed.fq /your/path/SRR8670768_2_trimmed.fq -o /your/path/bwa_mem_hg38.sam

#bwa for t2t
bwa mem -t 4 bwa_t2t_index /your/path/SRR8670768_1_trimmed.fq /your/path/SRR8670768_2_trimmed.fq -o /your/path/bwa_mem_t2t.sam
~                                                         
```



