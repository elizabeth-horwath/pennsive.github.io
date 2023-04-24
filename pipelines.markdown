
# PennSIVE pipelines

Generally we use R and develop R packages, but many of our tools have python and command-line interfaces as well.

If you're building a package that uses a PennSIVE tool, or need to fine-tune parameters not exposed by the wrappers, you're best off using the R package. If you're using python you can use a rpy2-based wrapper and be able to keep parameters that represent NIFTI images in memory. If you're building a neuroimaging pipeline in python with Nipype, we have a fork with our tools called PennSIVEpype. Otherwise, it's best to use the command-line wrappers.



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

As a nipype interface:
```py
from nipype.interfaces.mimosa import MIMoSA
mimosa = MIMoSA()
mimosa.inputs.t1 = "sub-01_mimosa-0.3.0/mimosa/t1_ws.nii.gz"
mimosa.inputs.flair = "sub-01_mimosa-0.3.0/mimosa/flair_ws.nii.gz"
mimosa.inputs.tissue = True
mimosa.inputs.verbose = True
mimosa.inputs.cores = 1
mimosa.run()
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

As a nipype interface:
```py
from nipype.interfaces.cvs import CVS
cvs = CVS()
cvs.inputs.t1 = "t1.nii.gz"
cvs.inputs.flair = "flair.nii.gz"
cvs.inputs.epi = "epi.nii.gz"
cvs.inputs.mimosa_prob_map = "mimosa_prob_map.nii.gz"
cvs.inputs.mimosa_bin_map = "mimosa_bin_map.nii.gz"
cvs.inputs.candidate_lesions = "candidate_lesions.npy"
cvs.inputs.cvs_prob_map = "prob_map.npy"
cvs.inputs.biomarker = "biomarker.npy"
cvs.inputs.parallel = False
cvs.inputs.skullstripped = True
cvs.inputs.biascorrected = True
cvs.inputs.c3d = True
cvs.inputs.cores = 1
cvs.run()
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

As a nipype interface:
```py
from nipype.interfaces.prls import PRL
prl = PRL()
prl.inputs.prob_map = "prob_map.nii.gz"
prl.inputs.lesion_map = "lesion_map.nii.gz"
prl.inputs.phase = "phase.nii.gz"
prl.inputs.disc = True
prl.run()
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

As a nipype interface:
```py
from nipype.interfaces.lesionclusters import LesionClusters
clusters = LesionClusters()
clusters.inputs.prob_map = "prob_map.nii.gz"
clusters.inputs.bin_map = "bin_map.nii.gz"
clusters.inputs.centers = "centers.nii.gz"
clusters.inputs.nnmap = "nnmap.nii.gz"
clusters.inputs.clusmap = "clusmap.nii.gz"
clusters.inputs.gmmmap = "gmmmap.nii.gz"
clusters.inputs.gmm = True
clusters.inputs.parallel = True
clusters.inputs.cores = 4
clusters.inputs.smooth = 1.2
clusters.inputs.min_center_size = 10
clusters.run()
```