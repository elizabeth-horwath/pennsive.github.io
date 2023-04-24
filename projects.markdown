# Project setup

## Setting up a project

Code and data should be version controlled with git and git annex, respectively. The changes code makes to data should be recorded with datalad, a powerful wrapper for git annex. Imaging data should be organized into BIDS, and curated with CuBIDS.

## Datalad
### Overview
Datalad can be installed with `conda create -n datalad -c conda-forge datalad`, then enabled with `conda activate datalad`. To create a new datalad repository, run `datalad create -c text2git -c yoda repository_name`. `-c text2git -c yoda` are datalad procedures that configure git annex and initialize your repository with a boilerplate directory structure.
```
├── CHANGELOG.md
├── README.md
└── code
    └── README.md
```

To version control data, run `datalad save -r -m "commit message"`. This will move your files into the "git annex" and replace them with symlinks from the annex. This keeps your data safe so if you accidentally `rm something`, you can restore it with `git checkout something`. Using `datalad save` is useful when initially moving source data into a repository, but is otherwise an anti-pattern since files will be created or modified by scripts, which you want to run through the `datalad run` wrapper. Although it is possible to run your scripts then save the results with `datalad save` afterwords, running your code with `datalad run` is preferred because it automatically checks to make sure inputs are available, unlocks and saves outputs, as well as commits the command line used to do the processing so results can be reproduced in an automated way. For example, instead of
```sh
datalad unlock my_inputs
./my_script.sh my_inputs my_outputs
datalad save -r -m "ran my script" my_outputs
```
do
```sh
datalad run -m "ran my script" -i my_inputs -o my_outputs ./my_script.sh "{inputs}" "{outputs}"
```
Once you've run your analysis with datalad run, make sure your results are reproducible by running `datalad rerun`. Code can be unreproducible in very non-obvious ways (for example, the default way of setting a seed won't work in R if your code is multithreaded), so running twice is the only way to make sure your analysis is reproducible.

### Caveats
Git can only handle so many files per repository, so it's important to break data into subdatasets, but there's also a practical limit on the number of subdatasets per dataset, so very large projects may need to restructure flat directory trees into many layers of nested subdatasets.

To create a subdataset, run `datalad create -d . subdataset` where `.` is the path to the parent dataset. Datalad uses git submodules to relate your parent dataset to its subdatasets, which record the path to the subdataset and which commit it's on. So when committing changes in subdatasets, you will need to make a commit in every parent dataset that you want to update the commit it's on (datalad save's `-r` flag is useful for this).

Since datalad monitors all files for changes parallelizing datalad run commands can be difficult as datalad doesn't know which files correspond to which runs. There are two workarounds. The simpler way is to use the `--explicit` flag to tell datalad to only monitor changes in the inputs and outputs provided by the `-i` and `-o` arguments. The more robust way is to checkout a new branch for every run command, and octopus merge them at the end.

<!-- Although most projects reside on network-mounted directories, datalads decentralized nature makes it easy to move files to a more performant filesystem (like `$TMPDIR`) then have results pushed back. For example, -->
### Example
First, create a repository, `analysis`, and setup a RIA store
```sh
mkdir my-project
cd my-project
PROJECTROOT=$PWD
input_store="ria+file://${PROJECTROOT}/input_ria"
output_store="ria+file://${PROJECTROOT}/output_ria"
# Create a source dataset with all analysis components as an analysis access point
datalad create -c yoda analysis
cd analysis
datalad create-sibling-ria -s output "${output_store}"
pushremote=$(git remote get-url --push output)
datalad create-sibling-ria -s input --storage-sibling off "${input_store}"
```
Then copy in and save your input data (assuming it's located at `$INPUTDATA`)
```sh
mkdir -p inputs/data
cp -r ${INPUTDATA}/* inputs/data
datalad save -r -m "added input data"
```
Then write a script that processes one subject or iteration in a temporary folder, (faster i/o and avoids conflict when running parallel datalad jobs).
```sh
#!/bin/bash

# fail whenever something is fishy, use -x to get verbose logfiles
set -e -u -x

ds_path=$(realpath $(dirname $0)/..) # assuming were in ./code
sub=$1
pushgitremote=$2
# $TMPDIR is a more performant local filesystem
# make sure to set this line depending on scheduler used!
wrkDir=$TMPDIR/$LSB_JOBID
# on SGE it's wrkDir=$TMPDIR/$JOB_ID
mkdir -p $wrkDir
cd $wrkDir
# get the output/input datasets
# flock makes sure that this does not interfere with another job
# finishing at the same time, and pushing its results back
# we clone from the location that we want to push the results too
# $DSLOCKFILE should be exported, it simply points to an empty file in .git used as a lock
flock $DSLOCKFILE datalad clone $ds_path ds
# all following actions are performed in the context of the superdataset
cd ds

# in order to avoid accumulation temporary git-annex availability information
# and to avoid a syncronization bottleneck by having to consolidate the
# git-annex branch across jobs, we will only push the main tracking branch
# back to the output store (plus the actual file content). Final availability
# information can be establish via an eventual `git-annex fsck -f joc-storage`.
# this remote is never fetched, it accumulates a larger number of branches
# and we want to avoid progressive slowdown. Instead we only ever push
# a unique branch per each job (subject AND process specific name)
git remote add outputstore "$pushgitremote"


# checkout new branches
# this enables us to store the results of this job, and push them back
# without interference from other jobs
git checkout -b "sub-${sub}"

# obtain datasets
datalad get inputs/data/sub-${sub}

# yay time to run
datalad run -i "inputs/data/sub-${sub}" -i "simg" -o "output" \
    singularity run -e \
    -B $TMPDIR \
    $PWD/simg/container_latest.sif \
    args to container

# file content first -- does not need a lock, no interaction with Git
datalad push --to output-storage
# and the output branch
flock $DSLOCKFILE git push outputstore
echo SUCCESS
cd ../..
chmod -R 777 $wrkDir
rm -rf $wrkDir
```
The above script is meant to be queued like
```sh
mkdir logs && echo logs >> .gitignore
export DSLOCKFILE=$PWD/.git/datalad_lock
touch $DSLOCKFILE
pushgitremote=$(git remote get-url --push output)
for sub in $(find inputs/data -type d -name 'sub-*' | cut -d '/' -f 3 ); do
    bsub -o logs ./code/project.sh $sub $pushgitremote
done
```
Then you can clone the output store anywhere by getting the dataset id `dsid=$(datalad -f '{infos[dataset][id]}' wtf -S dataset)` and cloning `datalad clone ria+file://$HOME/my-project/output_ria#${dsid} audit-my-project`.

For more information, the [datalad handbook](http://handbook.datalad.org/en/latest/index.html) is an excellent resource.
