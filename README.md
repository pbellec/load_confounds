# load_confounds

[![Pipy Badge](https://img.shields.io/pypi/v/load_confounds)](https://pypi.org/project/load-confounds/) [![Codacy Badge](https://api.codacy.com/project/badge/Grade/1da186ba5c44489b8af6d96a9c50d3c7)](https://app.codacy.com/gh/SIMEXP/fmriprep_load_confounds?utm_source=github.com&utm_medium=referral&utm_content=SIMEXP/fmriprep_load_confounds&utm_campaign=Badge_Grade_Dashboard) [![Maintainability](https://api.codeclimate.com/v1/badges/ce6f2bf20aa87accaaa4/maintainability)](https://codeclimate.com/github/SIMEXP/load_confounds/maintainability) [![CircleCI](https://circleci.com/gh/SIMEXP/load_confounds.svg?style=svg)](https://circleci.com/gh/SIMEXP/load_confounds) [![codecov](https://codecov.io/gh/SIMEXP/load_confounds/branch/master/graph/badge.svg)](https://codecov.io/gh/SIMEXP/load_confounds)

*Warning*: This package is still at an alpha stage of development. The API may still be subject to changes, and the library has not yet received much feedback from users.

## Quickstart

This package is used to easily load a sensible subset of variables from [fMRIprep](https://fmriprep.readthedocs.io/en/stable/) BOLD confounds in a python environment. `load_confounds` can be installed with `pip` (Python >=3.5) using:
```bash
pip install load_confounds
```
This example loads confounds using the `Params36`, i.e. the 36P denoising strategy of Ciric et al. 2017:
```python
from load_confounds import Params36
# load_confounds auto-detects the companion .tsv file (which needs to be in the same directory)
file = "path/to/file/sub-01_ses-001_bold.nii.gz"
confounds = Params36().load(file)
```
The `confounds` is a numpy ndarray ready to be plugged into a [nilearn](https://nilearn.github.io/) `masker`:
```python
img = masker.transform(file, confounds=confounds)
```

## Predefined denoising strategies
The predefined strategies are all adapted from Ciric et al. 2017, and currently include:
 * `Params2` : Mean white matter and CSF signals, with high-pass filter.
 * `Params6` : Basic motion parameters with high pass filter.
 * `Params9` : Basic motion parameters, WM/CSF signals, global signal and high pass filter.
 * `Params24` : Full motion parameters (derivatives, squares and squared derivatives), with high pass filter.
 * `Params36` : Motion parameters, WM/CSF signals, global signal, high pass filter. All noise components are fully expanded (derivatives, squares and squared derivatives).
 * `AnatCompCor` : Motion parameters (fully expanded), high pass filter, and acompcor.
 * `TempCompCor` : High pass filter, and tcompcor.

## Flexible denoising strategy
It is also possible to fine-tune a subset of noise variables, and the type of these variables:
```python
from load_confounds import Confounds
conf = Confounds(strategy=['high_pass', 'motion', 'global'], motion="derivatives")
confounds = conf.load('path/to/file/sub-01_ses-001.tsv')
```

Currently the following noise categories are supported:
* `motion` the motion parameters including 6 translation/rotation, and optionally derivatives, squares, and squared derivatives. Motion parameters can also be reduced through a PCA. Default: no PCA, and 24 motion model.
* `high_pass` basis of discrete cosines covering slow time drift frequency band.
* `wm_csf` the average signal of white matter and cerebrospinal fluid masks, and optionally derivatives, squares, and squared derivatives. Default: simple average, without squares or derivatives.
* `compcor` the results of a PCA applied on a mask based on either anatomy, temporal variance, or both. The number of components can be adjusted. Default: anat compcor with 6 components.
* ` global`  the global signal, and optionally derivatives, squares, and squared derivatives. Default: simple average without squares or derivatives.

See the docstring of `Confounds` for a list of available tweaking parameters.

## Nifti files and file collections
Note that if a `.nii.gz` file is specified, `load_confounds` will automatically look for the companion `tsv`confound file generated by fMRIprep. A pandas DataFrame can also be entered instead of a file name. It is also possible to specify a list of confound (or imaging) files, in which case `load_confounds` will return a list of numpy ndarray.

## A note on demeaning confounds
Unless you use the `detrend` or `high_pass` options of nilearn maskers, it may be important to demean the confounds. This is done by default by `load_confounds`, and is required to properly regress out confounds using nilearn with the `standardize=False`, `standardize=True` or `standardize="zscore"` options. If you want to use `standardize="psc"`, you will need to turn off the demeaning in `load_confounds`, which can be achieved using, e.g.:
```python
from load_confounds import Params6
conf = Params6(demean=False)
```

## Reference

Ciric R, Wolf DH, Power JD, Roalf DR, Baum GL, Ruparel K, Shinohara RT, Elliott MA, Eickhoff SB, Davatzikos C., Gur RC, Gur RE, Bassett DS, Satterthwaite TD. Benchmarking of participant-level confound regression strategies for the control of motion artifact in studies of functional connectivity. Neuroimage. 2017. doi:[10.1016/j.neuroimage.2017.03.020](https://doi.org/10.1016/j.neuroimage.2017.03.020)
