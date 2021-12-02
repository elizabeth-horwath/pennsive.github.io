---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
title: Welcome
nav_order: 1
description: "Welcome"
permalink: /
---
# Welcome to the Penn Statistics in Imaging and Visualization Endeavor (PennSIVE) Center
This is the internal website for PennSIVE tutorials and best practices.

If you just got cluster access, first setup your SSH keys:
```sh
ssh-keygen # if you've never done this or aren't sure; you can accept all the defaults, and don't need a passphrase
ssh-copy-id user@cluster
```
Then download VS code, docker, and read through the [environment setup guide](/environment-setup/).

When setting up a project meant for publication, read through the [project setup guide](/project-setup/) which explains how to version control code with git, track data provenance with datalad, and keep data organized according to the BIDS standard.

For more specific python neuroimaging tutorials, see the many [nipype examples](https://nipype.readthedocs.io/en/latest/examples.html). For more specific R neuroimaging tutorials, see [neuroconductor](https://neuroconductor.org/tutorials).

The [CBICA wiki](https://cbica-wiki.uphs.upenn.edu) (UPHS VPN required) is a good resource for CUBIC-specific tutorials, and [Ali's blog](https://www.alessandravalcarcel.com/blog/) offers lots of useful PMACS-specific information.

Also see the [PennLINC internal website](https://pennlinc.github.io) which offers many useful tutorials and tips.
