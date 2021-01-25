---
layout: page
title: Environment setup
permalink: /environment-setup/
nav_order: 2
---
# Setting up your environment
Neuroimaging software can be difficult to install and have complex dependencies. If any tool has a different version or is configured differently, results can become impossible to reproduce. Containers are like lightweight VMs that turns your environment into code so it can be shared, version controlled, and reproduced on any machine that supports a container runtime. On platforms where you have root access (like your computer), docker is the most convenient container runtime, but Singularity is a good alternative for rootless containers in cluster environments. 

There are some general purpose container images you can pull to do experimenting, but new projects with public results should create a project-specific container by writing a Dockerfile. NeuroDocker automates much of this process,
```
docker run --rm repronim/neurodocker:0.7.0 generate docker \
    --pkg-manager apt \
    --base debian:buster \
    --ants version=2.3.1 \
    --fsl version=6.0.3
```
which will write a Dockerfile to stdout (you can pipe directly into `docker build`) but any software they don't support has to be installed in that or another Dockerfile.

## Local development
Running a container with docker follows the format: `docker run [options for docker] image_name [payload process arguments]`. For example,
```
docker run -it -v $PWD:/data -w /data --rm pennsive/neuror:4.0 R -e "list.files()"
```
will run the pennsive/neuror container interactively (`-it`), setting up a bind mount (`-v`), setting the working directory for the payload process to that bind mount (`-w`), and remove the container after completion (`--rm`).

## Development on the cluster
The PMACS cluster is generally more amicable to interactive work while CUBIC is generally better for large batch and GPU jobs. Singularity is installed on both clusters, but on PMACS it's installed as an environment module so you need to run `module load DEV/singularity` first.

Singularity commands have different semantics than docker commands, but also follow the general format `singularity run [options for singularity] image_name [payload process arguments]`. The singularity equivalent of `-it` is `singularity shell` (instead of `singularity run`), instead of `-v` it's `-B`, instead of `-w` it's `--pwd`. Additionally, singularity by default does less to isolate the container than docker does, so you'll likely always need the `--cleanenv` or `--containall` flags to prevent host environment variables from overwriting those in the container. 

### PMACS
PMACS uses the LSF platform, which uses `bsub` to submit jobs. To submit an interactive job, run `bsub -Is -q "$QUEUE"_interactive 'bash';`. To run a non-interactive job, run `bsub -o /path/to/stdout -e /path/to/stderr ./my_job.sh`

### CUBIC
CUBIC uses the SGE platform, which uses `qsub` to submit jobs. There are no interactive compute nodes, but the login nodes are fairly beefy. To run a non-interactive job, run `qsub -o /path/to/stdout -e /path/to/stderr -b y -cwd -l h_vmem=16G -pe threaded 4-8 ./my_job.sh`. `-b y` tells SGE you're running a binary executable, `-cwd` makes the directory you issue the command from the working directory for the job, `-l h_vmem=16G` sets memory (default is only 4G!), and `-pe threaded 4-8` gives your job anywhere from 4-8 CPU cores depending on what the scheduler decides.

## GUIs
[coming soon...]

## Text editors
Generally, a text editor like VS Code or Atom provides the best user experience. However, R Studio images are available for local and remote development. Locally, you can start the R Studio server with
```
docker run --rm -d -e PORT=9090 -v $HOME:/data -w /data -p 80:9090 pennsive/rstudio:4.0
```
which will make R Studio available at [http://localhost:80](http://localhost).
On the cluster, you must first login with the `-L` option to create a ssh tunnel for the http traffic, then specify the port you tunneled when running the container
```
ssh -L12345:127.0.0.1:12345 user@cluster
SINGULARITYENV_PORT=12345 singularity run --cleanenv -B $TMPDIR:/var/run/rstudio-server /project/singularity_images/rstudio_4.0.sif
```
