---
layout: page
title: Nipype workflows
permalink: /workflows/
nav_order: 7
---
# Nipype workflows
Many neuroimaging tools have idiosyncratic CLIs that are clumsy to use from programming languages like python and R. Nipype provides a unified interface that facilitates the design of workflows within and between packages, lowering the learning curve necessary to use a new package.

Nipype interfaces consist of an input and output specification class, `_run_interface` method, and `_list_outputs`. To validate inputs are the requested type and outputs get created correctly, nipype uses a package called Traits to automate much of the process. When the `.run()` method is called on an instance of your interface, the `_run_interface` method is called, and `_list_outputs` is used to list the files matched against the output specification. For example, an interface that simply moves a file might look like
```py
class MoveResultFileInputSpec(BaseInterfaceInputSpec):
    in_file = File(exists=True, desc='input file to be renamed', mandatory=True)
    output_name = traits.String(desc='output name string')


class MoveResultFileOutputSpec(TraitedSpec):
    out_file = File(desc='path of moved file')


class MoveResultFile(BaseInterface):
    input_spec = MoveResultFileInputSpec
    output_spec = MoveResultFileOutputSpec

    def _run_interface(self, runtime):
        shutil.copyfile(self.inputs.in_file, self.inputs.output_name)
        return runtime

    def _list_outputs(self):
        outputs = self._outputs().get()
        outputs['out_file'] = self.inputs.output_name
        return outputs
```
A slightly more complex example, which thresholds an input image with nibabel might look like:
```py
from nipype.interfaces.base import BaseInterface, \
    BaseInterfaceInputSpec, traits, File, TraitedSpec
from nipype.utils.filemanip import split_filename

import nibabel as nb
import numpy as np
import os

class SimpleThresholdInputSpec(BaseInterfaceInputSpec):
    volume = File(exists=True, desc='volume to be thresholded', mandatory=True)
    threshold = traits.Float(desc='everything below this value will be set to zero',
                             mandatory=True)


class SimpleThresholdOutputSpec(TraitedSpec):
    thresholded_volume = File(exists=True, desc="thresholded volume")


class SimpleThreshold(BaseInterface):
    input_spec = SimpleThresholdInputSpec
    output_spec = SimpleThresholdOutputSpec

    def _run_interface(self, runtime):
        fname = self.inputs.volume
        img = nb.load(fname)
        data = np.array(img.get_data())

        active_map = data > self.inputs.threshold

        thresholded_map = np.zeros(data.shape)
        thresholded_map[active_map] = data[active_map]

        new_img = nb.Nifti1Image(thresholded_map, img.affine, img.header)
        _, base, _ = split_filename(fname)
        nb.save(new_img, base + '_thresholded.nii')

        return runtime

    def _list_outputs(self):
        outputs = self._outputs().get()
        fname = self.inputs.volume
        _, base, _ = split_filename(fname)
        outputs["thresholded_volume"] = os.path.abspath(base + '_thresholded.nii')
        return outputs
```

When writing workflows, most of the time a tool will already have an interface, see the nipype [interfaces index](https://nipype.readthedocs.io/en/latest/interfaces.html) for a complete list. A workflow will typically begin by parsing command line arguments
```py
parser = argparse.ArgumentParser()
parser.add_argument('-i', '--input', type=str, default='T1W.nii.gz')
parser.add_argument('-o', '--output', type=str, default='/tmp/tmp')
args = parser.parse_args()

```
setup a workflow and interface node:
```py
wf = Workflow('threshold')

thresh_phase = Node(SimpleThreshold(), 'thresh_phase')
thresh_phase.inputs.volume = args.input
thresh_phase.inputs.threshold = 0.5
```
then another node, connected to the first
```py
move_phase = Node(MoveResultFile(), 'move_phase')
move_phase.inputs.output_name = args.output
wf.connect([(thresh_phase, move_phase, [('out_file', 'input_image')])])
```
<!-- ## ANTS
## FSL
## Freesurfer -->

<!-- talk about using nipype through reticulate -->
