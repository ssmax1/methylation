# BISCUIT WGBS Pipeline  
*A workflow for whole‑genome bisulfite sequencing*

This repository contains a portable pipeline for processing whole‑genome bisulfite sequencing (WGBS) data using BISCUIT, Skewer, Samtools, and related tools.  
All configuration is handled through a single `config.env` file, making the workflow easy to adapt to any SLURM‑based HPC environment.

---

## Repository structure

```text
methylation/BISCUIT/
├── runBISCUIT           # Main SLURM pipeline script
├── BISCUIT_config.env   # User-editable configuration file
└── README.md            # This file
```

---

## Requirements

### SLURM-based HPC system
The pipeline requires SLURM for job submission.

### Software
Tools may be provided via modules or absolute paths in `config.env`:

- BISCUIT
- samtools
- samblaster
- skewer
- pigz
- GNU parallel
- bedtools
- bgzip / tabix (HTSlib)

### Reference genome
The script automatically downloads if missing:

- GRCh38 primary assembly FASTA
- GRCh38 GTF annotation

You may replace URLs in `config.env` to use any genome.

### Input FASTQs
Paired‑end FASTQs must follow this pattern:

sample_*1.fq.gz  
sample_*2.fq.gz

Split‑lane files must share the same prefix.

---

## Configuration

All user‑specific settings live in BISCUIT_config.env.

Edit this file before running the pipeline:

nano config.env

Key settings include:

- Project directories
- Reference genome URLs
- Adapter file
- SLURM resources (THREADS, MEMORY)
- Email for notifications
- Tool paths (if not using modules)

This design ensures the pipeline script itself never needs modification.

---

## Running the pipeline

### 1. (Optional) Load your config

source config.env

### 2. Submit the job

sbatch run_biscuit.sh

### 3. Monitor progress

squeue -u $USER

Logs are written to:

$WDIR/logs/

---

## Pipeline workflow

The pipeline performs the following steps automatically:

### 1. Reference preparation
- Downloads FASTA + GTF
- Unzips
- Builds BISCUIT index

### 2. FASTQ preparation
- Identifies unique sample prefixes
- Merges split FASTQs (or symlinks if only one pair exists)

### 3. Trimming
Performed using Skewer with adapters defined in config.env.

### 4. Alignment
Pipeline:  
biscuit align → samblaster → samtools sort

### 5. QC
- BAM index
- Flagstats
- BISCUIT QC script

### 6. Methylation extraction
Produces:

- .vcf.gz
- .bed.gz
- Biseq‑format .bed
- methylKit‑format .mk.bed

Each step creates a .complete file to allow safe re‑runs.

---

## Output files

For each sample:

sample.bisc.bam  
sample.bisc.bam.bai  
sample.bisc.bam.flagstats  
sample.vcf.gz  
sample.bed.gz  
sample.bed  
sample.mk.bed  
sample.qc/

---

## Customization

You can modify:

- Reference genome URLs
- Adapter sequences
- SLURM resources
- Tool paths
- QC behavior
- Array size (#SBATCH --array=1-N%M)

All via config.env.
