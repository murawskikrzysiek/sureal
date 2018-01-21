SUREAL - Subjective Recovery Analysis
===================

SUREAL is a toolbox developed by Netflix for recovering quality scores from noisy measurements obtained by subjective tests. Read [this](resource/doc/dcc17v3.pdf) paper for some background.

## Prerequisites & Installation

SUREAL requires a number of Python packages pre-installed:

  - [`numpy`](http://www.numpy.org/) (>=1.12.0)
  - [`scipy`](http://www.scipy.org/) (>=0.17.1)
  - [`matplotlib`](http://matplotlib.org/1.3.1/index.html) (>=2.0.0)
  - [`pandas`](http://pandas.pydata.org/) (>=0.19.2)
  - [`scikit-learn`](http://scikit-learn.org/stable/) (>=0.18.1)

First, upgrade `pip` to the newest version:

```
sudo -H pip install --upgrade pip
```

Then install the required Python packages:

```
pip install --user numpy scipy matplotlib pandas scikit-learn
```

Add the `python/src` subdirectories to the environment variable `PYTHONPATH`:

```
export PYTHONPATH="$(pwd)/python/src:$PYTHONPATH"
```

You can also add it to the environment permanently, by appending to `~/.bashrc`:

```
echo export PYTHONPATH="$(pwd)/python/src:$PYTHONPATH" >> ~/.bashrc
source ~/.bashrc
```

Under macOS, use `~/.bash_profile` instead.

## Testing

The package has thus far been tested on Ubuntu 16.04 LTS and macOS 10.13. After installation, run:

```
./unittest
```

## Usage in Command Line

Under root directory, run `./run_subj` to print usage information:

```
usage: run_subj subjective_model dataset_filepath 
```

Below are two example usages:

```
./run_subj MLE resource/dataset/NFLX_dataset_public_raw_last4outliers.py
./run_subj MLE_CO resource/dataset/VQEGHD3_dataset_raw.py
```

Here `subjective_model` are the available subjective models offered in the package, including:
  - MOS - Standard mean opinion score
  - MLE - Full maximum likelihood estimation (MLE) model that takes into account both subjects and contents
  - MLE_CO - MLE model that takes into account only subjects ("Content-Oblivious")
  - DMOS - Differential MOS, as defined in [ITU-T P.910](https://www.itu.int/rec/T-REC-P.910)
  - DMOS_MLE - apply MLE on DMOS
  - DMOS_MLE_CO - apply MLE_CO on DMOS
  - SR_MOS - Apply subject rejection (SR), as defined in [ITU-R BT.500](https://www.itu.int/rec/R-REC-BT.500), before calculating MOS
  - ZS_SR_MOS - Apply z-score transformation, followed by SR, before calculating MOS
  - SR_DMOS - Apply SR, before calculating DMOS
  - ZS_SR_DMOS - Apply z-score transformation, followed by SR, before calculating DMOS

`dataset_filepath` is the path to a dataset file. There are two ways to construct a dataset file. The first way is only useful when the subjective test is full sampling, i.e. every subject views every distorted video. For example:

```
from vmaf.config import VmafConfig
ref_videos = [
    {'content_id': 0, 'content_name': 'checkerboard', 'path': 
        VmafConfig.test_resource_path('yuv', 'checkerboard_1920_1080_10_3_0_0.yuv')},
    {'content_id': 1, 'content_name': 'flat', 'path': 
        VmafConfig.test_resource_path('yuv', 'flat_1920_1080_0.yuv')},
]
dis_videos = [
    {'content_id': 0, 'asset_id': 0, 'os': [100, 100, 100, 100, 100], 'path': 
        VmafConfig.test_resource_path('yuv', 'checkerboard_1920_1080_10_3_0_0.yuv')},
    {'content_id': 0, 'asset_id': 1, 'os': [40, 45, 50, 55, 60],  'path': 
        VmafConfig.test_resource_path('yuv', 'checkerboard_1920_1080_10_3_1_0.yuv')},
    {'content_id': 1, 'asset_id': 2, 'os': [90, 90, 90, 90, 90],  'path': 
        VmafConfig.test_resource_path('yuv', 'flat_1920_1080_0.yuv')},
    {'content_id': 1, 'asset_id': 3, 'os': [70, 75, 80, 85, 90],  'path': 
        VmafConfig.test_resource_path('yuv', 'flat_1920_1080_10.yuv')},
]
ref_score = 100
```

In this example, `ref_videos` is a list of reference videos. Each entry is a dictionary, and must have keys `content_id`, `content_name` and `path` (the path to the reference video file). `dis_videos` is a list of distorted videos. Each entry is a dictionary, and must have keys `content_id` (the same content ID as the distorted video's corresponding reference video), `asset_id`, `os` (stands for "opinion score"), and `path` (the path to the distorted video file). The value of `os` is a list of scores, reach voted by a subject, and must have the same length for all distorted videos (since it is full sampling). `ref_score` is the score assigned to a reference video, and is required when differential score is calculated, for example, in DMOS.

The second way is more general, and can be used when the test is full sampling or partial sampling (i.e. not every subject views every distorted video). The only difference from the first way is that, the value of `os` is now a dictionary, with the key being a subject ID, and the value being his/her voted score for particular distorted video. For example:

```
'os': {'Alice': 40, 'Bob': 45, 'Charlie': 50, 'David': 55, 'Elvis': 60}
```

Since partial sampling is allowed, it is not required that every subject ID is present in every `os` dictionary.

## Usage in a Python Script

More examples of using the subjective models in a Python script can be found in [/python/script/run_subjective_models.py](/python/script/run_subjective_models.py). Run the script first to get a sense of what it does.