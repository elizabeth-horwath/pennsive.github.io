---
layout: page
title: Environment setup
permalink: /environment-setup/
nav_order: 2
---
# Setting up your environment
Neuroimaging software can be difficult to install and have complex dependencies. If any tool has a different version or is configured differently, results can become impossible to reproduce. Containers are like lightweight VMs that turns your environment into code so it can be shared, version controlled, and reproduced on any machine that supports a container runtime. On platforms where you have root access (like your computer), docker is the most convenient container runtime, but Singularity is a good alternative for rootless containers in cluster environments. 

There are some general purpose container images you can pull to do experimenting, but new projects with public results should create a project-specific container by writing a Dockerfile. NeuroDocker automates much of this process,
```sh
docker run --rm repronim/neurodocker:0.7.0 generate docker \
    --pkg-manager apt \
    --base debian:buster \
    --ants version=2.3.1 \
    --fsl version=6.0.3
```
which will write a Dockerfile to stdout (you can pipe directly into `docker build`) but any software they don't support has to be installed in that or another Dockerfile.

---

## Local development

### Docker
Running a container with docker follows the format: `docker run [options for docker] image_name [payload process arguments]`. For example,
```sh
docker run -it -v $PWD:/data -w /data --rm pennsive/neuror:4.0 R -e "list.files()"
```
will run the pennsive/neuror container interactively (`-it`), setting up a bind mount (`-v`), setting the working directory for the payload process to that bind mount (`-w`), and remove the container after completion (`--rm`). [Download](https://docs.docker.com/docker-for-mac/install/) docker desktop for mac.

### Visual Studio Code
[Download](https://code.visualstudio.com/download) vs code for mac, then install all the essential extensions with
```sh
which code && for extension in $(curl -s https://raw.githubusercontent.com/PennSIVE/pennsive.github.io/main/vs-code-extension-list.txt); do
code --install-extension $extension
done || echo "in the vscode command pallet (command + shift + p) search for \"Install code command in 'PATH'\""
```

### Startup scripts
Clone the lab's bash/zsh [startup scripts](https://github.com/PennSIVE/bash), which creates some convenient aliases
```sh
cd ~/repos
git clone https://github.com/PennSIVE/bash.git
cd bash
cp .env.sample .env
vim .env # edit as necessary
# open your ~/.bashrc or ~/.zshrc and add:
if [ -f $HOME/repos/bash/.bashrc ]; then
    source $HOME/repos/bash/.bashrc
fi
```
Try creating a SSH tunnel for HTTP traffic (`takimhttp` or `cbicahttp`) then starting R studio with `rstudio` (which should then be available at [http://localhost](http://localhost)).

### Other tools
[homebrew](https://brew.sh), [starship](https://starship.rs), [ITK-SNAP](http://www.itksnap.org/pmwiki/pmwiki.php?n=Downloads.SNAP3)

---

## Development on the cluster
The PMACS cluster is generally more amicable to interactive work while CUBIC is generally better for large batch and GPU jobs. Singularity is installed on both clusters, but on PMACS it's installed as an environment module so you need to run `module load DEV/singularity` first.

Singularity commands have different semantics than docker commands, but also follow the general format `singularity run [options for singularity] image_name [payload process arguments]`. The singularity equivalent of `-it` is `singularity shell` (instead of `singularity run`), instead of `-v` it's `-B`, instead of `-w` it's `--pwd`. Additionally, singularity by default does less to isolate the container than docker does, so you'll likely always need the `--cleanenv` or `--containall` flags to prevent host environment variables from overwriting those in the container. 

### PMACS
PMACS uses the LSF platform, which uses `bsub` to submit jobs. To submit an interactive job, run `bsub -Is -q "$QUEUE"_interactive 'bash';`. To run a non-interactive job, run `bsub -o /path/to/stdout -e /path/to/stderr ./my_job.sh`

### CUBIC
CUBIC uses the SGE platform, which uses `qsub` to submit jobs. There are no interactive compute nodes, but the login nodes are fairly beefy. To run a non-interactive job, run `qsub -o /path/to/stdout -e /path/to/stderr -b y -cwd -l h_vmem=16G -pe threaded 4-8 ./my_job.sh`. `-b y` tells SGE you're running a binary executable, `-cwd` makes the directory you issue the command from the working directory for the job, `-l h_vmem=16G` sets memory (default is only 4G!), and `-pe threaded 4-8` gives your job anywhere from 4-8 CPU cores depending on what the scheduler decides.
