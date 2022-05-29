# Mesmerize Intro
**If you are using the VM, first upgrade it to v0.7.2 if you have not done so already:**

v0.7.2 contains back-ported fixes from the current `master` branch and works on python3.6 and within the VM. It requires CaImAn v1.8.8

```bash
# activate the environment
source ~/venvs/mesmerize/bin/activate
# get the latest version of mesmerize
pip install --upgrade mesmerize
```

For other installation methods I recommend installing from `master` on GitHub (requires python>=3.8) or v0.7.2 from PYPI.

## Launch Mesmerize:
VM: double click icon, or (better) open terminal, call `mesmerize`

Other methods: activate your python venv or conda env, call `mesmerize`

## Welcome Window
Docs page: http://docs.mesmerizelab.org/en/master/user_guides/general/misc.html

### System Configuration
Docs page: http://docs.mesmerizelab.org/en/master/user_guides/general/misc.html#system-configuration

**Check your workdir, NO SPACES**

#### Check pre-run commands

##### Windows
The default setting should work
The BLAS & MKL settings should be automatic from the CaImAn installation
Uncomment conda activation command, make sure it activates the right env
Add the conda activiation command if it doesn't already exist


##### Linux
```bash
source /path/to/env/bin/activate
export OPENBLAS_NUM_THREADS=1
export MKL_NUM_THREADS=1
```

##### Mac
make sure you have:

```
source activate <conda_env_name>
```

The BLAS & MKL settings should be automatic from the CaImAn installation

## Viewer
Docs page: http://docs.mesmerizelab.org/en/master/user_guides/viewer/overview.html

Click the Viewer button in the Welcome Window 

