# Mesmerize Intro
Download the demo data: https://www.dropbox.com/s/r3nzryz35mmm44f/mesmerize-demos.zip?dl=0

**If you are using the VM, first upgrade it to v0.7.2 if you have not done so already:**

v0.7.2 contains back-ported fixes from the current `master` branch and works on `python3.6` and within the VM. It requires **CaImAn v1.8.8**

```bash
# activate the environment
source ~/venvs/mesmerize/bin/activate
# get the latest version of mesmerize
pip install --upgrade mesmerize
```

For other installation methods I recommend installing from `master` on GitHub (requires `python>=3.8`) or v0.7.2 from PYPI.

## Launch Mesmerize:
VM: double click icon, or (better) open terminal, call `mesmerize`

Other methods: activate your python venv or conda env, call `mesmerize`

## Welcome Window
Docs page: http://docs.mesmerizelab.org/en/master/user_guides/general/misc.html

### System Configuration
Docs page: http://docs.mesmerizelab.org/en/master/user_guides/general/misc.html#system-configuration

**Check your workdir, NO SPACES**

#### Check pre-run commands

#### Windows
The default setting should work
The BLAS & MKL settings should be automatic from the CaImAn installation
Uncomment conda activation command, make sure it activates the right env
Add the conda activiation command if it doesn't already exist


#### Linux
```bash
source /path/to/env/bin/activate
export OPENBLAS_NUM_THREADS=1
export MKL_NUM_THREADS=1
```

#### Mac
make sure you have:

```
source activate <conda_env_name>
```

The BLAS & MKL settings should be automatic from the CaImAn installation

## Viewer
Docs page: http://docs.mesmerizelab.org/en/master/user_guides/viewer/overview.html

Click the Viewer button in the Welcome Window 

Open an image file using the Tiff file module
Docs: http://docs.mesmerizelab.org/en/master/user_guides/viewer/modules/tiff_file.html

Overview of pqytgraph ImageView widget:
  
* Scrolling through video
* Histogram to change min-max
* Change cmap
* Pan/zoom
* Measure tool

# Interace with CaImAn

## Batch Manager

### Intro
Docs: http://docs.mesmerizelab.org/en/master/user_guides/viewer/modules/batch_manager.html

Why:
  * Parameter optimization, notebooks are cumbersome when you have tens of videos, and need to search the parameter space
    * Hyperparameter gridsearch
    * Set to run various parameter combinations, run overnight, come back and see what worked
  * File organization
    * Use pandas DataFrame to interface with intput data, parameters, and output files
    * More with Eric later

### Create batches with the GUI
**MCorr**

1. Open raw movie
2. Open MCorr GUI
3. Enter param variant
4. Add to batch
5. Enter a few more param variants and add to batch
6. Run batch
7. Explore results
    * Scroll through MCorr movie
    * Mean projections of single movies
    * Mean projections of multiple movies (slow when n_movies is small).
    * Look at shifts, crispness etc. with Eric later

**CNMF with MCorr output**
1. Open CNMF GUI, enter some param variants, add to batch.
2. Run batch
3. Explore results
    * ROI Manager, view single components, multiple components
    * Run DFOF
    * View `cnmf.estimates.C`, DFOF and spikes `cnmf.estimates.S`

**CNMFE**

Same as above but with endoscope demo and using `gSig_filt` for mcorr
  
### Batches with scripts

**This is the most efficient way to use Mesmerize**

#### Simple Viewer API Examples

Open the Viewer Console

Load a movie

```python
path = "/path/to/movie.tiff"
meta_path = path[:-5] + ".json"

# Create a new work environment with this image sequence
work_env = ViewerWorkEnv.from_tiff(path, "imread", meta_path)

# set it as the current work environment
vi.viewer.workEnv = work_env
vi.update_workEnv()
```

Load a movie and set meta data from a dict: http://docs.mesmerizelab.org/en/master/user_guides/viewer/modules/tiff_file.html#console-script-usage
  
```python
image_path = # path to tiff file

clear_workEnv() # Prevents a confirmation dialog from appearing

# Get the tiff module
tio = get_module('tiff_io', hide=True)

# Load the tiff file
# this is a higher level interface than calling `ViewerWorkEnv.from_tiff()`
tio.load(image_path, method='imread', axes_order='txy')

meta_dict = \
    {
        "origin":   "my_microscope_software",   # must a str
        "fps":      17.25,                      # must be a int or float
        "date"      "20201123_172345"           # must be a str formatted as "YYYYMMDD_HHMMSS"/
    }

get_workEnv().imgdata.meta = meta_dict
```

#### Script Editor

Same example as above but using the script editor

#### Motion Correction
  
```python
# some mcorr params
mc_kwargs = \
{
    "max_shifts":           (6, 6),
    "niter_rig":            2,
    "max_deviation_rigid":  3,
    "strides":              (196, 196),
    "overlaps":             (98, 98),
    "upsample_factor_grid": 4,
    "gSig_filt":            (10, 10)  # Set to `None` for 2p data
}

params = \
{
    'mc_kwargs':        mc_kwargs,  # the kwargs we set above
    'item_name':        "will set later per file",
    'output_bit_depth': "Do not convert"  # can also set to `8` or `16` if you want the output in `8` or `16` bit
}

# Open a tiff file as shown before

# Get caiman motion correction module, hide=False to not show GUI
mc_module = get_module("caiman_motion_correction", hide=True)

# Set name for this video file
name = os.path.basename(path)[:-5]
params["item_name"] = name

# First variant of params
params["mc_kwargs"]["strides"] = (196, 196)
params["mc_kwargs"]["overlaps"] = (98, 98)

# Add one variant of params for this video to the batch
mc_module.add_to_batch(params)

# Try another variant of params
params["mc_kwargs"]["strides"] = (256, 256)
params["mc_kwargs"]["overlaps"] = (128, 128)

# Set these params and add to batch
mc_module.add_to_batch(params)

# Try one more variant of params
params["mc_kwargs"]["strides"] = (296, 296)
params["mc_kwargs"]["overlaps"] = (148, 148)

# Set these params and add to batch
mc_module.add_to_batch(params)
```
#### Gridsearch with a directory of files

Motion correction: http://docs.mesmerizelab.org/en/master/user_guides/viewer/modules/caiman_motion_correction.html

CNMF: http://docs.mesmerizelab.org/en/master/user_guides/viewer/modules/cnmf.html

CNMFE: http://docs.mesmerizelab.org/en/master/user_guides/viewer/modules/cnmfe.html
