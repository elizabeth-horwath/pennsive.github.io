---
layout: page
title: Lab pipelines
permalink: /pennsive-pipelines/
nav_order: 5
---
# PennSIVE pipelines
## MIMoSA
MIMoSA automates multiple sclerosis (MS) lesion segmentation with just a T1w and FLAIR required.

> [github.com/avalcarcel9/mimosa](https://github.com/avalcarcel9/mimosa)

Available as a [BIDS app](https://github.com/PennSIVE/mimosa), on [docker hub](https://hub.docker.com/r/pennsive/mimosa). For example:
```sh
# singularity pull docker://pennsive/mimosa
singularity run --cleanenv --bind ${PWD} --bind ${TMPDIR} \
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

If your data are not in BIDS it's possible to run the CLI of mimosa directly on a T1/FLAIR pair by using `docker run --entrypoint=""` instead `docker run` or `singularity exec` instead of `singularity run` and then calling /run.R directly. For example:
```sh
singularity exec --cleanenv --bind ${PWD} --bind ${TMPDIR} \
        /path/to/image /run.R \
        -i /path/to/folder/with/niftis \
        -o /path/to/outdir \
        -f flair.nii.gz \
        -t t1.nii.gz \
        -s mass \
        --n4 \
        --whitestripe
```

## CVS
The central vein sign (CVS) uses a lesion probability map (from MIMoSA) and a vessellness filtering process to find veins running through lesions, biomarker of MS.

> [github.com/jdwor/cvs](https://github.com/jdwor/cvs)

Available as a [BIDS app](https://github.com/PennSIVE/cvs), on [docker hub](https://hub.docker.com/r/pennsive/cvs). For example:
```sh
singularity run --cleanenv --bind ${PWD} --bind ${TMPDIR} \
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


## PRL
The presence of paramagnetic rims around lesions (PRL) is another MS biomarker.

> [github.com/carolynlou/prlr](https://github.com/carolynlou/prlr)

Available as a [BIDS app](https://github.com/PennSIVE/prls), on [docker hub](https://hub.docker.com/r/pennsive/prls). For example:
```sh
singularity run --cleanenv --bind ${PWD} --bind ${TMPDIR} \
        /path/to/image \
        inputs/data \
        outputs \
        participant \
        --participant_label $sub_num
```

## Center detection
From the lesiontools package, the lesion center detection method helps count distinct lesions.

> [github.com/jdwor/lesiontools](https://github.com/jdwor/lesiontools)

Available as a [BIDS app](https://github.com/PennSIVE/lesiontools), on [docker hub](https://hub.docker.com/r/pennsive/lesionclusters). For example:
```sh
singularity run --cleanenv --bind ${PWD} --bind ${TMPDIR} \
        /path/to/image \
        inputs/data \
        outputs \
        participant \
        --participant_label $sub_num \
        --strip mass \
        --n4 \
        --min-center-size 10 \
        --gmm \
        --skip_bids_validator
```