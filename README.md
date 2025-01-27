# Part 1: Download and Prepare All the Data

### Dependencies
- Git clone pipeline: [bact-gen-scripts](https://github.com/sanger-pathogens/bact-gen-scripts)
- Conda environment: `bact-gen-scripts.yml`

### Step 1.1: SRA Download using Accession Number from Their Supplemental Material Table 1
```bash
sbatch /storage/home/hcoda1/2/bli629/scratch/redo_paper/download_parallel.sbatch
```

### Step 1.2: Zip All the Short Reads
```bash
sbatch /storage/home/hcoda1/2/bli629/scratch/original_paper/all_codes/gz_01.sbatch
```

### Step 1.3: Short Reads Quality Control using `fastp`
```bash
sbatch /storage/home/hcoda1/2/bli629/scratch/original_paper/all_codes/fastp_01.sbatch
```

### Step 1.4: Use Pipeline `bact-gen-scripts` to Get the Alignments Mapping Against Reference Genome for Each Sample
```bash
sbatch /storage/home/hcoda1/2/bli629/scratch/original_paper/all_codes/map_to_bam_original.sbatch
```

### Step 1.5: Identify the Alignments with Low Quality
```bash
python /storage/home/hcoda1/2/bli629/scratch/original_paper/all_codes/filter_percentage.py
```
<br>
<br>

# Part 2: Clone Assignment

### Dependencies
- Conda environment: `clone_assignment.yml`

### Step 2.1: Determine the Samples with Unique Strain-Type-Patient Pair as Representative for Following SNP Distance Clustering
Result accession numbers are in `accession_unique_pair.txt`:
```bash
python /storage/home/hcoda1/2/bli629/scratch/original_paper/all_codes/unique_ST_Patient_pair.py
```

### Step 2.2: Combine the Alignments for the Samples with Unique Strain-Type-Patient Pair and Run Through `snp-sites` to Extract SNPs
1. Combine the alignments:
   ```bash
   xargs -I {} -a /path/to/accession_unique_pair.txt cat /path/to/bact-gen/step1.4/output/folder/{}/{}_qc.mfa > /path/to/output/merged.aln
   ```
2. Run `snp-sites`:
   ```bash
   snp-sites /path/to/output/merged.aln -o /path/to/output/merged_snpsites.aln
   ```
3. Rename the `.aln` to `.fasta`:
   ```bash
   cp /path/to/output/merged_snpsites.aln /path/to/output/merged_snpsites.fasta
   ```

### Step 2.3: Run `pairsnp` to Obtain Pairwise SNP Distance Matrices
```bash
pairsnp -t 12 /path/to/output/merged_snpsites.aln > /path/to/pairsnp/output.csv
```

### Step 2.4: UPGMA Clustering to Cluster the Samples Based on the SNP Distance Matrix
```bash
python /storage/home/hcoda1/2/bli629/scratch/original_paper/all_codes/UPGMA_clustering.py
```

After clustering the representative samples, the cluster results were reflected to all the samples using the R script below:
```bash
/storage/home/hcoda1/2/bli629/scratch/original_paper/all_codes/cluster.R
```
<br>
<br>

# Part 3: Mutational Burden Analysis Workflow

### Dependencies
- Conda environment: `phyloeffects.yml`, `snpeff.yml`

### Step 3.1: Use Alignment Files and Phylogenetic Trees to Reconstruct the Ancestral Genome for Each Clone
All the data are downloaded from [Zenodo](https://zenodo.org/records/10600286):
```bash
sbatch /storage/home/hcoda1/2/bli629/scratch/original_paper/all_codes/phylloeffects.sbatch
```

### Step 3.2: Extract the Reconstructed Ancestral Genome
This step is incorporated with Step 3.

### Step 3.3: Compare the Reconstructed Ancestral Genome with the Reference Genome at Each Mapped Position
Variants are identified and saved to VCF files:
```bash
sbatch /storage/home/hcoda1/2/bli629/scratch/original_paper/all_codes/reconstruction_and_vcf_building.sbatch
```

### Step 3.4: Annotate the Output VCF Files Using `SnpEff`
- The `SnpEff` database was manually built using the GenBank file from the [NCBI Reference Genome](https://www.ncbi.nlm.nih.gov/nuccore/AE004091.2/?&withparts=on&expand-gaps=on).
- Follow the instructions from the [SnpEff documentation](https://pcingola.github.io/SnpEff/snpeff/build_db/).
- Database name was set as: `GCA_000006765.1_ASM676v1_genomic`
```bash
sbatch /storage/home/hcoda1/2/bli629/scratch/original_paper/all_codes/snpEff.sbatch
```

### Step 3.5: Conduct Mutational Burden Analysis
```bash
python /storage/home/hcoda1/2/bli629/scratch/original_paper/all_codes/mutational_burden_analysis.py
```

### Step 3.6: Generate Manhattan Plot Visualization
Use the R script provided for visualization:
```bash
/storage/home/hcoda1/2/bli629/scratch/original_paper/all_codes/manhattan_plot.R
```
