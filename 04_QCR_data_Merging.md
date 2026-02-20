## Rehead all VCFs

#!/bin/bash
#PBS -N rehead_files
#PBS -l walltime=20:00:00
#PBS -l select=1:ncpus=5:mem=20GB
#PBS -o /home/n11565608/0_Project/PROJECT_DATA/Obj3_QCR_Data_and_analysis/01_Data/02_Rehead_All_VCFs/01_Script/01_reheader_all_vcfs.out
#PBS -e /home/n11565608/0_Project/PROJECT_DATA/Obj3_QCR_Data_and_analysis/01_Data/02_Rehead_All_VCFs/01_Script/01_reheader_all_vcfs.err
#PBS -M anoop.joseph@hdr.qut.edu.au
#PBS -m abe
#PBS -V



# load modules
module load GCC/13.2.0
module load BCFtools/1.19


#!/usr/bin/env bash
set -euo pipefail

# Folder with your VCFs (edit if needed)
VCF_DIR="/home/n11565608/0_Project/PROJECT_DATA/Obj3_QCR_Data_and_analysis/01_Data/bcf_unzip/bcf-anoop"
OUT_DIR="/home/n11565608/0_Project/PROJECT_DATA/Obj3_QCR_Data_and_analysis/01_Data/02_Rehead_All_VCFs/02_Reheaded_data"

mkdir -p "$OUT_DIR"

shopt -s nullglob
vcfs=("$VCF_DIR"/*.vcf.gz)

if [ ${#vcfs[@]} -eq 0 ]; then
  echo "No .vcf.gz files found in: $VCF_DIR"
  exit 1
fi

echo "Found ${#vcfs[@]} VCFs"

for vcf in "${vcfs[@]}"; do
  base=$(basename "$vcf")
  new_id="${base%.vcf.gz}"  # use filename as sample ID

  # Get current sample name (should be exactly 1 for single-sample VCF)
  old_id=$(bcftools query -l "$vcf" | head -n 1)

  if [ -z "${old_id:-}" ]; then
    echo "ERROR: Could not read sample name from $vcf"
    continue
  fi

  # Sanity: ensure it's single-sample
  n_samples=$(bcftools query -l "$vcf" | wc -l | tr -d ' ')
  if [ "$n_samples" -ne 1 ]; then
    echo "WARNING: $vcf has $n_samples samples (expected 1). Skipping."
    continue
  fi

  out_vcf="$OUT_DIR/${new_id}.vcf.gz"

  # Create a sample rename map: old<TAB>new
  mapfile=$(mktemp)
  printf "%s\t%s\n" "$old_id" "$new_id" > "$mapfile"

  echo "Reheader: $base  |  $old_id  ->  $new_id"

  # Reheader and write compressed VCF
  bcftools reheader -s "$mapfile" -o "$out_vcf" "$vcf"

  # Index
  tabix -f -p vcf "$out_vcf"

  rm -f "$mapfile"
done












## 03a_patch_filter_headers_v2.pbs
#!/bin/bash
#PBS -N patch_vcf_filters_v2
#PBS -l walltime=08:00:00
#PBS -l select=1:ncpus=5:mem=20GB
#PBS -j oe
#PBS -o /home/n11565608/0_Project/PROJECT_DATA/Obj3_QCR_Data_and_analysis/01_Data/02_Rehead_All_VCFs/01_Script/03a_patch_filters_v2.log
#PBS -M anoop.joseph@hdr.qut.edu.au
#PBS -m abe
#PBS -V

set -euo pipefail

module load GCC/13.2.0
module load BCFtools/1.19
module load HTSlib/1.19

# Use the already-filterfixed directory as input (or go back to original reheaded dir if you prefer)
IN_DIR="/home/n11565608/0_Project/PROJECT_DATA/Obj3_QCR_Data_and_analysis/01_Data/02_Rehead_All_VCFs/02_Reheaded_data_filterfixed"
OUT_DIR="/home/n11565608/0_Project/PROJECT_DATA/Obj3_QCR_Data_and_analysis/01_Data/02_Rehead_All_VCFs/02_Reheaded_data_filterfixed_v2"
mkdir -p "$OUT_DIR"

# Add a broader set so you don't keep hitting this.
FILTER_HDR=$(cat <<'EOF'
##FILTER=<ID=VQSRTrancheSNP99.00to99.90,Description="VQSR tranche SNP 99.00-99.90 (source pipeline)">
##FILTER=<ID=VQSRTrancheSNP99.90to100.00,Description="VQSR tranche SNP 99.90-100.00 (source pipeline)">
##FILTER=<ID=VQSRTrancheINDEL99.00to99.90,Description="VQSR tranche INDEL 99.00-99.90 (source pipeline)">
##FILTER=<ID=VQSRTrancheINDEL99.90to100.00,Description="VQSR tranche INDEL 99.90-100.00 (source pipeline)">
##FILTER=<ID=bad_mapping_loci,Description="Flagged as bad mapping locus (source pipeline)">
EOF
)

shopt -s nullglob
vcfs=("$IN_DIR"/*.vcf.gz)

if [ ${#vcfs[@]} -eq 0 ]; then
  echo "ERROR: No .vcf.gz files found in $IN_DIR"
  exit 1
fi

echo "Found ${#vcfs[@]} VCFs"

for vcf in "${vcfs[@]}"; do
  base=$(basename "$vcf")
  out="$OUT_DIR/$base"

  echo "Patching header: $base"

  tmp_hdr=$(mktemp)
  bcftools view -h "$vcf" > "$tmp_hdr"

  # Only patch once (check for one of the keys)
  if ! grep -q "##FILTER=<ID=VQSRTrancheINDEL99.00to99.90" "$tmp_hdr"; then
    awk -v add="$FILTER_HDR" '
      BEGIN{printed=0}
      /^#CHROM/ && printed==0 {print add; printed=1}
      {print}
    ' "$tmp_hdr" > "${tmp_hdr}.new"
    mv "${tmp_hdr}.new" "$tmp_hdr"
  fi

  bcftools reheader -h "$tmp_hdr" -o "$out" "$vcf"
  tabix -f -p vcf "$out"

  rm -f "$tmp_hdr"
done

echo "Done. Output in: $OUT_DIR"












## 03b_merge_multisample_vcf_v2.pbs

#!/bin/bash
#PBS -N merge_vcfs_v2
#PBS -l walltime=10:00:00
#PBS -l select=1:ncpus=20:mem=80GB
#PBS -j oe
#PBS -o /home/n11565608/0_Project/PROJECT_DATA/Obj3_QCR_Data_and_analysis/01_Data/02_Rehead_All_VCFs/01_Script/03b_merge_v2.log
#PBS -M anoop.joseph@hdr.qut.edu.au
#PBS -m abe
#PBS -V

set -euo pipefail

module load GCC/13.2.0
module load BCFtools/1.19
module load HTSlib/1.19

IN_DIR="/home/n11565608/0_Project/PROJECT_DATA/Obj3_QCR_Data_and_analysis/01_Data/02_Rehead_All_VCFs/02_Reheaded_data_filterfixed_v2"
OUT_DIR="/home/n11565608/0_Project/PROJECT_DATA/Obj3_QCR_Data_and_analysis/01_Data/02_Rehead_All_VCFs/03_MultiSample_VCF"
mkdir -p "$OUT_DIR"

MERGED_VCF="${OUT_DIR}/QCR_1526_merged.multisample.vcf.gz"
VCF_LIST="${OUT_DIR}/vcf_list_fixed_v2.txt"

shopt -s nullglob
vcfs=("$IN_DIR"/*.vcf.gz)

if [ ${#vcfs[@]} -eq 0 ]; then
  echo "ERROR: No .vcf.gz files found in $IN_DIR"
  exit 1
fi

printf "%s\n" "${vcfs[@]}" > "$VCF_LIST"

echo "Merging $(wc -l < "$VCF_LIST") VCFs..."

while read -r f; do
  [ -f "${f}.tbi" ] || tabix -f -p vcf "$f"
done < "$VCF_LIST"

bcftools merge \
  -l "$VCF_LIST" \
  -Oz \
  -o "$MERGED_VCF" \
  -m none \
  --force-samples \
  --threads 10

tabix -f -p vcf "$MERGED_VCF"

echo "Number of samples in merged VCF:"
bcftools query -l "$MERGED_VCF" | wc -l

echo "Done: $MERGED_VCF"






echo "Done. Output in: $OUT_DIR"

