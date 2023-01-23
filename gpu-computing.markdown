---
layout: page
title: GPU Computing
permalink: /gpu-computing/
nav_order: 10
---
# GPU Computing


## 1. Logging onto the cluster

To use the GPU machines on cluster, you need to log in to cluster. Here is the nice introduction of how to log in: `https://www.alessandravalcarcel.com/blog/2019-04-23-ssh/`

`lpcgpu01`, which is the host for GPU machines, is accessible via `takim` server. You may enter the `takim` server by typing this:

```sh
ssh -X <pennkey>@takim.pmacs.upenn.edu
```

*** Don't read this if you are not familiar with GPU computing ***

For advanced user, looking for additional GPU power, we have six additional GPU cores, exclusively available to us. `takim2` is a submit host (not a excutible host) that can be used for GPU computing. You can access it by typing this:

```sh
ssh -X <pennkey>@takim2.pmacs.upenn.edu
```
or
```sh
ssh -X <pennkey>@takim2
```

Note that this is a submit host, not a executable host. You can directly use it as a interactive session, but can't submit a normal job to `takim2`.


## 2. Interactive Session Basics

If you intend to use an interactive session, consider using screen so that you don't lose your work. You can open a screen by

```sh
screen -S <Screen-name>
```

and start your work. You can find the details about how to use a screen at: `https://www.alessandravalcarcel.com/blog/2019-06-12-interactivesession1/`

Once you are in the executable host, you can open an interactive session by typing this:

```sh
bsub -Is -q lpcgpu -gpu "num=1" -n 1 "bash"
```

Make sure you request gpu. "num=1" requests the number of GPU whereas "-n 1" requests the number of CPU.

Next, load torch and tensorflow to activate CUDA

```sh
module load torch
module load tensorflow/2.3-GPU
```

To check whether your CUDA is running, run this:

```sh
python
```

In Python, run this:

```py
import torch
torch.cuda.is_available()
torch.cuda.device_count()
torch.cuda.current_device()
torch.cuda.device(0)
torch.cuda.get_device_name(0)
```

The response should be
```py
torch.cuda.is_available() : True
torch.cuda.device_count() : 1
torch.cuda.current_device() : 0
torch.cuda.device(0) : <torch.cuda.device object at 0x2ae68db7cb20>
torch.cuda.get_device_name(0) : 'NVIDIA GeForce RTX 2080 Ti'
```

Now, you are ready to use the GPU!


## 3. Normal Job Sessions

Once you are in the executable host, you can submit a normal job, usually with the bash file.

```sh
bsub -q lpcgpu -gpu "num=1" -n 1 -J "orig[1-3]" -o <where to save your log file> <location of your bash file>
```
For example, I save my log file at `/home/ecbae/nnUNet.txt` and bash file at `/home/ecbae/orig.sh`. Note that my job index is [1-3], which in result requests 3 GPU cores and 3 CPU cores.

My bash file looks like this:

```sh
module load torch
module load tensorflow/2.3-GPU

nnUNet_plan_and_preprocess -t $(( $LSB_JOBINDEX +149 ))
nnUNet_train 2d nnUNetTrainerV2 $(( $LSB_JOBINDEX +149 )) 0 --npz
nnUNet_train 2d nnUNetTrainerV2 $(( $LSB_JOBINDEX +149 )) 1 --npz
nnUNet_train 2d nnUNetTrainerV2 $(( $LSB_JOBINDEX +149 )) 2 --npz
nnUNet_train 2d nnUNetTrainerV2 $(( $LSB_JOBINDEX +149 )) 3 --npz
nnUNet_train 2d nnUNetTrainerV2 $(( $LSB_JOBINDEX +149 )) 4 --npz
```

Note that we have 10 GPU cores in `lpcgpu01` host. If you need more GPU cores, you may want to use the `takim2` host, discussed in the first section.

Also, I am running a pre-installed python package `nnUNet`. To install the existing package, you should submit a ticket to PMACS or send an email to Martin Das.


## 4. Concluding remarks

We need more GPU!
