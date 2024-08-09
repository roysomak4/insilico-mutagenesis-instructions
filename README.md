# Instructions to perform Insilico Mutagenesis
This document describes a step by step process to perform insilico mutagenesis using the InSim bioinformatics tool
https://github.com/thesushantpatil/insiM

1. Start with an aligned BAM file to a known version of the human reference genome. In this example, we will use GRCh37 human reference genome. This BAM file should be derived from sequencing a real sample or a well characterized reference material. This repository does not provide the BAM file. Please use a BAM file from your laboratory or publicaly available source.
2. This BAM file should be coordinate-sorted and indexed. If the index is missing, it can be generated using the following command `samtools index sample1.bam`
3. Prepare BED file to define the variants of interest for mutagenesis. An example format is provided below
```
13	32913711	32913711	del;0.52;;4
11	108121753	108121753	del;0.52;;2
11	108183151	108183151	snv;0.52;T;
2	215595181	215595181	ins;0.52;CATACTTTTCTTCCTGTTCA;
17	59761414	59761414	del;0.52;;4
22	29130389	29130389	snv;0.52;T;
```
More information about this specification can be found at the InSim Github repo. https://github.com/thesushantpatil/insiM/blob/master/insiM_example_data/Hybrid_capture/targets_capture.bed

4. Prepare the configuration file for InSim to initiate insilico mutagenesis. An example configuration file is provided below
   ```
    # Mandatory args
    assay = capture
    bam = sample1.bam
    target = variants_of_interest.bed
    mutation = mix
    genome = reference/GRCh37.fa

    # Conditional/optional arguments.
    ampliconbed =
    read =
    vaf =
    len =
    seq =
    out = sample1
   ```
For additional details please refer to the InSim GitHub repo.

5. Run the insilico mutagenesis algorithm
   ```
   python insiM_v1.0.py sample1_config.txt
   ```
6. The output consists of a pair of FASTQ files and a VCF file containing the details of the inserted variants. For example, `sample1_R1_001.fastq`, `sample1_R2_001.fastq`, and `sample1.insiM.vcf`


#### Quality Control to confirm the variants of interest have been successfully mutagenized
1. Align the FASTQ files from step 6 to the same human reference genome that was used for aligning the input BAM file (step 1).
   ```
   bwa mem -M -v 1 -t 10 -R '@RG\tID:sample1\tSM:sample1\tPL:ILLUMINA\tPI:150\tCN:lab1' reference/GRCh37.fa sample1_R1_001.fastq.gz sample1_R2_001.fastq.gz | samtools sort -@ 10 -o sample1_sorted.bam 
   ```  
2. Index BAM file
   ```
   samtools index sample1_sorted.bam
   ```
3. Mark PCR duplicates
   ```
   sambamba markdup -t 10 -p sample1_sorted.bam sample1_dedup.bam
   ```
4. Call variants
   ```
   vardict -G reference/GRCh37.fa -f 0.01 -r 4 -o 1.5 -th 30 -L 200 -N sample1 -b sample1_dedup.bam -c 1 -S 2 -E 3 -g 4 | vardict_app/bin/teststrandbias.R | vardict_app/bin/var2vcf_valid.pl -A -N sample1 -E -f 0.01 >sample1_raw.vcf
   ```