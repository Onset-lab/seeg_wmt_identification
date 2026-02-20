# Documentation: SEEG White Matter Tracts Identification

## Overview
This repository contains scripts for analyzing stereoelectroencephalography (SEEG) white matter stimulation data. It provides tools to generate bipolar stimulated regions of interest (ROI) and analyze their intersection with white matter bundles.

## Prerequisites
- **onsetpy**: Installation required from [Onset-lab/onsetpy](https://github.com/Onset-lab/onsetpy)
- **scilpy**: Installation required from [scilus/scilpy](https://github.com/scilus/scilpy)
- **SurgeryFlow**: Must be executed prior to using these scripts. Available at [Onset-lab/SurgeryFlow](https://github.com/Onset-lab/SurgeryFlow)
- **Electrode masks**: Must be registered to T1 preoperative anatomical space

## Main Procedures

### 1. Generate Bipolar Stimulated ROI
Creates regions of interest based on bipolar electrode stimulation within white matter.

**Command:**

```bash
onset_get_bundles_in_bipolar_electrodes \
    --bundles export_preop/Clean_Bundles \
    --wm export_preop/wm.nii.gz \
    --electrodes electrodes_preop/
```

### 2. Filter Bundles by Stimulated ROI
Intersect the generated ROI with white matter bundles to quantify overlap.

**Command:**

```bash
for j in electrodes_preop/vat*; do
    for i in export_preop/Clean_Bundles/*.trk; do
        output=$(scil_tractogram_filter_by_roi.py "${i}" tmp.trk \
            --drawn_roi "$j" any include --display_count -f 2>/dev/null)
        
        result=$(echo "$output" | tr "'" '"' | jq -r '
            if .streamline_count_final_filtering > 0 then
                "\(.streamline_count_final_filtering) \(.streamline_count_before_filtering)"
            else
                empty
            end' 2>/dev/null)
        
        if [ -n "$result" ]; then
            read final before <<< "$result"
            ratio=$(echo "scale=4; $final / $before * 100" | bc)
            echo "Bundle: $(basename "$i") VAT: $(basename "$j") | Perc: $ratio %"
        fi
    done
done
```

This script filters each bundle by the VAT (Volume of Activated Tissue) ROI and calculates the percentage of streamlines intersecting the stimulated region.
