---
layout: page
title: Lab pipelines
permalink: /pennsive-pipelines/
nav_order: 5
---
# Lab pipelines
## MIMoSA
MIMoSA automates multiple sclerosis (MS) lesion segmentation with just a T1w and FLAIR required.
> [github.com/PennSIVE/mimosa](https://github.com/PennSIVE/mimosa)
To use as a BIDS app:
```sh
singularity run --cleanenv -B ${PWD} -B ${TMPDIR} \
        /path/to/image \
        inputs/data \
        outputs \
        participant \
        --participant_label $sub_num \
        --strip mass \
        --n4 \
        --register \
        --whitestripe \
        --thresh 0.25 \
        --debug \
        --skip_bids_validator
```

To use as a datalad bootstrap:
```sh
curl -LO https://raw.githubusercontent.com/PennLINC/TheWay/main/scripts/pmacs/bootstrap-mimosa.sh
bash bootstrap-mimosa.sh --bids-input ria+file:///path/to/bids  --container-ds /path/or/uri/to/containers
cd mimosa/analysis
bash code/bsub_calls.sh
# when jobs complete
bash code/merge_outputs.sh
```

## CVS
The central vein sign (CVS) uses a lesion probability map (from MIMoSA) and a vessellness filtering process to find veins running through lesions, biomarker of MS.
> [https://github.com/PennSIVE/cvs](https://github.com/PennSIVE/cvs)
To use as a BIDS app:
```sh
singularity run --cleanenv -B ${PWD} -B ${TMPDIR} \
        /path/to/image \
        inputs/data \
        outputs \
        participant \
        --participant_label $sub_num \
        --skullstrip \
        --n4 \
        --thresh 0.25 \
        --skip_bids_validator
```
## PRL (coming soon)
<!-- The presence of paramagnetic rims around lesions (PRL) is a MS biomarker -->
