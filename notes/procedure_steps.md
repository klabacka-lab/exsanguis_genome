# A. exsanguis assembly – procedure log


## Raw data:
  - Run 1: /grphome/grp_reptile_genomics/exsanguis_genome/daniel_lizard/m84100_241113_062539_s3.hifi_reads.bc1017.fastq.gz (FASTQ)
  - Run 2: /grphome/grp_reptile_genomics/exsanguis_genome/7889/m84100_250911_183557_s3.hifi_reads.bc1015.bam (BAM, needs converting)
 

## Step 1: Convert run 2 BAM to FASTQ (the first run data is already in that format)
Run: sbatch jobs/01_bam2fastq.job
What it does: turns the BAM into a compressed FASTQ so hifiasm can use it
Output: ~/nobackup/autodelete/exsanguis/raw_fastq/m84100_250911_183557_s3.hifi_reads.bc1015.fastq.gz

## Step 2: Read stats
Run: sbatch jobs/02_read_stats.job
What it does: checks the FASTQs are complete and gives read numbers
Output: results/read_stats.tsv

## Step 3: Genome assembly (hifiasm)
- Run: sbatch jobs/03_hifiasm.job
- What it does: assembles the genome from both runs as a triploid (3 copies)
- Output: ~/nobackup/autodelete/exsanguis/hifiasm/aexs.asm.bp.p_utg.fasta (all copies) and aexs.asm.bp.p_ctg.fasta (one collapsed copy)
  
  **hifiasm command**
- `-o aexs.asm`: prefix (start of the name) for every output file
- `-t 48`: use 48 threads (matches `--ntasks`)
- `--n-hap 3`: the genome has 3 copies (triploid); default is 2
- `$RUN1 $RUN2`: the two read files to assemble together
  **awk lines (GFA → FASTA)**
- hifiasm saves results as `.gfa`, a graph format with different line types:
  - `S` lines = the actual sequences (contig name + DNA)
  - other lines (`L`, `A`, ...) = how pieces connect and which reads built them
- `/^S/`: only look at lines that start with `S`
- `print ">"$2"\n"$3`: print `>` + the name (column 2), then the DNA (column 3) on the next line
- `> ....fasta`: save it as a FASTA file
- Result: same sequences, just in FASTA format (only names + DNA), which most other tools need. The DNA itself is not changed.
- Line 1 converts **p_ctg** (one collapsed copy), line 2 converts **p_utg** (all 3 copies)
