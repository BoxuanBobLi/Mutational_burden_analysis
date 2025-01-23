```markdown
# Mutational Burden Analysis Workflow

### Step 1: Use alignment files and phylogenetic trees to reconstruct the ancestral genome for each clone
```bash
sbatch /storage/home/hcoda1/2/bli629/scratch/original_paper/code_management/phylloeffects.sbatch
```

### Step 2: Extract the reconstructed ancestral genome
Details for this step should be implemented as required.

### Step 3: Compare the reconstructed ancestral genome with the reference genome at each mapped position. Variants are identified and saved to VCF files.
```bash
sbatch /storage/home/hcoda1/2/bli629/scratch/original_paper/code_management/reconstruction_and_vcf_building.sbatch
```

### Step 4: Annotate the output VCF files using SnpEff
- The SnpEff database was manually built using the GenBank file from [NCBI Reference Genome](https://www.ncbi.nlm.nih.gov/nuccore/AE004091.2/?&withparts=on&expand-gaps=on).
- Follow the instructions from the [SnpEff documentation](https://pcingola.github.io/SnpEff/snpeff/build_db/).
- Database name was set as: `GCA_000006765.1_ASM676v1_genomic`
```bash
sbatch /storage/home/hcoda1/2/bli629/scratch/original_paper/code_management/snpEff.sbatch
```

### Step 5: Conduct mutational burden analysis
```bash
python /storage/home/hcoda1/2/bli629/scratch/original_paper/code_management/mutational_burden_analysis.py
```

### Step 6: Generate Manhattan plot visualization
Use the R script provided for visualization.
```bash
/storage/home/hcoda1/2/bli629/scratch/original_paper/code_management/manhattan_plot.R
```
```
