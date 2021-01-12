---
layout: page
title: Project organization
permalink: /bids/
---

# BIDS
When labs organize datasets in different ways time is often wasted rewriting scripts to expect a particular structure. BIDS solves this problem by standardizing the organization of neuroimaging data. BIDS is nothing more than a prescribed directory structure and file naming convention, but it allows labs to more easily share datasets and processing pipelines, interact with the dataset programmatically via pybids, and run all the most popular neuroimaging tools ("BIDS Apps") in a consistent way.

Although it's possible to manually organize NIFTIs into the BIDS structure, doing so is error-prone and inefficient. Manual organization may be the only option if dicoms aren't available, but otherwise you can use heudiconv to automatically format the produced NIFTIs according to BIDS.

Unlike `dcm2niix` which can simply be fed a directory of dicoms, heudiconv requires a bit more setup. First, your dicoms be organized either by StudyUID or accession_number. Then, you need to run heudiconv with the `-c none -f convertall` options to generate TSVs with dicom metadata needed to write a heuristic. Generally, the protocol_name will be the most (only?) useful field, and is used in the heuristic to determine which dicoms go together to create a NIFTI.

For example,
```
heudiconv \
-d "/path/to/data/{subject}/{session}/*.dcm" \
    -o /path/to/output -f convertall -s 001 -ss abc -c none -b -g accession_number
# there will now be a hidden .heudiconv directory with dicom header TSVs
# once you've determined which dicoms go where, write a heuristic then use the -c dcm2niix option to tell heudiconv to do the actual dcm2niix conversion
heudiconv \
-d "/path/to/data/{subject}/{session}/*.dcm" \
    -o /path/to/output -f /path/to/your_heuristic.py -s 001 -ss abc -c dcm2niix -b -g accession_number
```

Beyond organizing data with BIDS, [datalad](/reproducibility) provides good recommendations for organizing code. The [YODA](https://handbook.datalad.org/en/latest/basics/101-127-yoda.html) procedure will initialize your project with this directory structure:
```
├── CHANGELOG.md
├── README.md
└── code
    └── README.md
```
A more comprehensive directory structure might look like this:
```
├── ci/                         # continuous integration configuration
│   └── .travis.yml
├── code/                       # your code
│   ├── tests/                  # unit tests to test your code
│   │   └── test_myscript.py
│   └── myscript.py
├── docs                        # documentation about the project
│   ├── build/
│   └── source/
├── envs                        # computational environments
│   └── Singularity
├── inputs/                     # dedicated inputs/, will not be changed by an analysis
│   └─── data/
│       ├── dataset1/           # one stand-alone data component
│       │   └── datafile_a
│       └── dataset2/
│           └── datafile_a
├── outputs/                    # outputs away from the input data
│   └── important_results/
│       └── figures/
├── CHANGELOG.md                # notes for fellow humans about your project
├── HOWTO.md
└── README.md
```