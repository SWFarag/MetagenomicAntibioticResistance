# NastyBugs

### A Simple Method for Extracting Antimicrobial Resistance Information from Metagenomes
##### Hackathon team: Lead: Steve Tsang - SysAdmins: Greg Fedewa, Daniel Quang, Sherif Farag - Writers: Matthew Moss, Alexey V. Rakov

Antibiotic resistance of bacterial pathogens remains a major threat to public health around the world. Fast and reliable extraction of antimicrobial resistance genomic signatures from large raw sequencing datasets obtained from human metagenomes is a key task for bioinformatics. **NastyBugs** is a versatile workflow for fast extracting of antimicrobial resistance genomic signatures from metagenomes.

*Objective*: Create a reusable, reproducible, scalable, interoperable workflow 
to locate antimicrobial resistant genomic signatures in SRA shotgun sequencing (metagenomics) datasets

## Dependencies:computer:

*Software:*

[Magic-BLAST 1.3b](https://github.com/boratyng/magicblast)

[SAMtools 1.3.1](http://www.htslib.org/)

[FASTX-Toolkit](http://hannonlab.cshl.edu/fastx_toolkit/)

[Docker](https://www.docker.com/)

*DBs used for BLAST databases:*

[NCBI GRCh37/UCSC hg19 human reference genome](https://www.ncbi.nlm.nih.gov/projects/genome/guide/human/index.shtml)

[CARD (Comprehensive Antibiotic Resistance Database) DB](https://card.mcmaster.ca/)

[RefSeq Reference Bacterial Genomes](https://www.ncbi.nlm.nih.gov/refseq/)

## Workflow

![My image](https://github.com/NCBI-Hackathons/MetagenomicAntibioticResistance/blob/master/AbxResistanceMetagenomics.png)

## Workflow method

The pipeline use three databases that should be downloaded with the script:
1.	**GRCh37/hg19 human reference genome database** used for alignment and filtering reads of human origin from metagenomics samples.
2.	**CARD database** used for search of genomic signatures in the subset of reads unaligned to human genome.
3.	**RefSeq reference bacterial genomes database** used for search and assigning of 16S RNA taxonomic labels the subset of reads unaligned to human genome.

Step 1.  Mapping sample SRR to human genome using Magic-BLAST:
```
>magicblast13 -sra SRRXXXXXXX -db ~/references/human -num_threads 12 -score 50 -penalty -3 -out ~/test_run/SRRXXXXXXX_human.sam
```

Step 2. Filtering reads mapped to human genome using SAMtools (Removal of host (human) genome from metagenomics data):
```
>samtools fasta -f 4 SRRXXXXXXX_human.sam -1 SRRXXXXXXX_read1.fasta  -2 SRRXXXXXXX_read2.fasta -0 SRRXXXXXXX_read0.fasta
>fastx_clipper [-i INFILE] [-o OUTFILE]
```

Step 3. Searching 16S RNA taxonomic labels in RefSeq reference bacterial genomes database to identify microbial species presented in metagenome using Magic-BLAST:
```
>magicblast13 -infmt fasta -query ~/test_run/SRRXXXXXXX_read1.fasta -query_mate ~/test_run/SRRXXXXXXX_read2.fasta -num_threads 12 -score 50 -penalty -3 -out ~/test_run/SRRXXXXXXX_refseq.sam -db ~/references/REFSEQ
```

Step 4. Searching genes and SNPs from CARD database in metagenome using Magic-BLAST:
```
>magicblast13 -infmt fasta -query ~/test_run/SRRXXXXXXX_read1.fasta -query_mate ~/test_run/SRRXXXXXXX_read2.fasta -num_threads 12 -score 50 -penalty -3 -out ~/test_run/SRRXXXXXXX_CARD_SNP.sam -db ~/references/CARD_variant
>magicblast13 -infmt fasta -query SRRXXXXXXX_read1.fasta -query_mate SRRXXXXXXX_read2.fasta -num_threads 12 -score 50 -penalty -3 -out SRRXXXXXXX_CARD_gene.sam -db ~/references/CARD_gene
```

Step 5. Converting SAM to BAM format and sorting using SAMtools:
```
>samtools view -bS SRRXXXXXXX_SNP.sam | samtools sort - -o SRRXXXXXXX_SNP.bam
>samtools view -bS SRRXXXXXXX_CARD_gene.sam | samtools sort - -o SRRXXXXXXX_CARD_gene.bam
```

Step 6. Producing detailed output file(s) including names of detected bacterial species and resistance genes with statistical metrics in text and graphical formats.

## Deliverables

Documented workflow with containerized tools in Docker

## Installation
```
sudo docker images
sudo docker pull stevetsa/docker-magicblast
sudo docker run -it stevetsa/docker-magicblast
sudo docker ps -a 
```

## Usage

main.sh <options> -S SRA -o output_directory

## Input file format

SRA accession numbers (ERR or SRR)
or
FASTQ files

## Output

1. Table (in CSV or TAB-delimited format) with the next columns:
- RefSeq accession number (Nucleotide)
- Genus
- Resistance gene
- ARO (Antibiotic Resistance Ontology)
- Score (number of mapped reads per 1kb)

2. Dot plot showing relative abundance of antimicrobial resistance/bacterial species in metagenomic sample.

3. Pie chart vizualization of bacterial abundance in the given dataset (Ondov BD, Bergman NH, and Phillippy AM. Interactive metagenomic visualization in a Web browser. BMC Bioinformatics. 2011 Sep 30; 12(1):385).
![My image](https://github.com/NCBI-Hackathons/MetagenomicAntibioticResistance/blob/master/MetagenomeVisualization.png)

## Warnings

## Planned Features

## F.A.Q.

## People/Team
* Steve Tsang, NCI/NIH, Gaithersburg, MD, <tsang@mail.nih.gov>
* Greg Fedewa, UCSF, San Francisco, CA, <greg.fedewa@gmail.com>
* Sherif Farag, UNC, Chapel Hill, NC, <farags@email.unc.edu>
* Matthew Moss, CSHL, Cold Spring Harbor, NY, <moss@cshl.edu>
* Daniel Quang, UCI, Irvine, CA, <dxquang@uci.edu>
* Alexey V. Rakov, UPenn, Philadelphia, PA, <rakovalexey@gmail.com>

