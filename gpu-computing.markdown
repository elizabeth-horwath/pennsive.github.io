---
layout: page
title: GPU Computing
permalink: /gpu-computing/
nav_order: 10
---
# GPU Computing


## 1. Logging onto the cluster

Nice intro `https://www.alessandravalcarcel.com/blog/2019-04-23-ssh/`

```sh
ssh -X <pennkey>@takim.pmacs.upenn.edu
```

```sh
ssh -X <pennkey>@takim2.pmacs.upenn.edu
```

```sh
ssh -X <pennkey>@takim2
```


## 2. Interactive Session Basics

If you intend to use an interactive session, consider using screen so that you don't lose your work. You can open a screen by

```sh
screen -S <Screen-name>
```

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

```py
import torch
torch.cuda.is_available()
torch.cuda.deevice_count()
torch.cuda.current_device()
torch.cuda.device(0)
torch.cuda.geet_device_name(0)
```

## 3. Batch Jobs Sessions

## 4. Commands and References



If that does not work then you need to install homebrew

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Then, install Ruby and the Bundler gem.

```sh
brew install ruby
gem install bundler --user-install
```

Now, make sure you're inside `pennsive.github.io` and run

```sh
bundle install
```

This will look for a Gemfile and install the necessary dependencies. Once that's done you can test the website! Run `bash run.sh` and on a browser go to `http://127.0.0.1:4000/`. Voilà your own local copy of the PennSIVE wiki!

## 4. Work on your contribution article

Now we're ready to make some changes. Open your favorite text editor and create an empty file with `.markdown` as extension. Give it a descriptive name!
Then add a YAML section to the top, which will tell Jekyll how to render the page. For instance, you may edit the YAML of this page

```sh
---
layout: page
title: Contributing to Wiki
permalink: /contributing/
nav_order: 9
---
```

Write your article with the usual markdown syntax. You may look at the other `.markdown` files for examples.

Note: If run `bash run.sh` and add changes while that process is running, you may see those changes by refreshing your browser.

# 5. Push to your

Once you're happy with your progress you may push to your own repository
