![Heteromotility logo](logo.png)

## Introduction

`Heteromotility` is a tool for analyzing cell motility in a quantitative manner. `Heteromotility` takes timelapse imaging data as input and calculates 70+ 'motility features' that can be used to generate a 'motility fingerprint' for a given cell. By analyzing more features of cell motility than most common cell tracking methods, `Heteromotility` may be able to identify novel heterogenous motility phenotypes.

`Heteromotility` also contains a suite of tools to quantify and visualize cell state spaces, and dynamic state transitions within the state space. While these tools were developed for use with `Heteromotility` features, they may be applied to any arbitrary time-series feature set. These tools are implemented modularly, separate from the core `Heteromotility` feature extraction package.

Heteromotility is developed by [Jacob Kimmel](http://jacobkimmel.github.io/) in the [Brack](http://www.bracklab.com/) and [Marshall](http://biochemistry2.ucsf.edu/labs/marshall/) Labs at the [University of California, San Francisco](http://www.ucsf.edu/).

We've posted a pre-print applying `Heteromotility` analysis to quantify dynamic cell state transitions in muscle stem cells and a cancer cell model. Check it out on bioRxiv!  

[Inferring cell state by quantitative motility analysis reveals a dynamic state system and broken detailed balance](http://www.biorxiv.org/content/early/2017/07/26/168534.article-info)

## Installation

Heteromotility can be installed from the Python Package Index with **pip**

    $ pip install heteromotility

or

Simply clone this repository and run **setup.py** in the standard manner.  

    $ git clone https://github.com/jacobkimmel/heteromotility
    $ cd heteromotility
    $ python setup.py install
    $ heteromotility -h
    usage: Calculate motility features from cell locations or cell paths
    [-h] [--seg SEG] [--exttrack EXTTRACK] input_dir output_dir
    ...

Both methods of package installation will add an alias `heteromotility` to your `PATH` environment variable.

As with all research software, we recommend installing `Heteromotility` inside of a [virtual environment](https://virtualenv.pypa.io/en/stable/).

    $ virtualenv heteromotility_env/
    $ source heteromotility_env/bin/activate
    $ pip install heteromotility

## Motility Features

Feature Name | Notation | Description
-------------|----------|--------------
Total Distance | total_distance | total distance traveled
Net Distance | net_distance | net distance traveled
Minimum Speed | min_speed | minimum overall speed
Maximum Speed | max_speed | maximum speed
Average Speed | avg_speed | average overall speed
Time Spent Moving [1,10] | time_moving | proportion of time in motion, variable time intervals considered [1,10] frames
Average Moving Speed [1,10] | avg_moving_speed | average speed during movement, variable time intervals considered [1,10] frames
Linearity | linearity | Pearson's *r*<sup>2</sup> of regression through all cell positions
Spearman's rho<sup>2</sup> | spearmanrsq | Spearman's rho<sup>2</sup> through all cell positions
Progressivity | progressivity | (net distance traveled / total distance traveled)
Mean Square Displacement Coefficient | msd_alpha | coefficient of log(tau) in a plot of log(tau) vs log(MSD)
Random Walk Linearity Comparison | rw_linearity | (cell path linearity - simulated random walk linearity)
Random Walk Net Distance Comparison | rw_net_distance | (cell path net distance - simulated random walk net distance)
Random Walk Kurtosis Comparison [1,10] | diff_kurtosis | (cell displacement kurtosis - random walk kurtosis) for variable time intervals
Hurst Exponent (Rescaled-Range) | hurst_RS | Hurst exponent estimation using Mandelbrot's rescaled range methods
Autocorrelation [1,10] | autocorr | Autocorrelation of the displacement distribution for variable time lags
Non-Gaussian coefficient | nongauss | Non-Gaussian coefficient (alpha_2) of the displacement distribution
Proportion of Right Turns [time_lag, estimation_interval] | p_turn | proportion of turns an object makes to the left, calculated for multiple time lags and direction estimation intervals
Minimum Turn Magnitude | min_theta | minimum turn angle in radians, for various time lags and direction estimation intervals
Maximum Turn Magnitude | max_theta | maximum turn angle in radians, for various time lags and direction estimation intervals
Average Turn Magnitude | mean_theta | mean turn angle in radians, for various time lags and direction estimation intervals

# Usage

Heteromotility is invoked as a command line tool from your terminal of choice. For the following examples, it is assumed that the alias `heteromotility` has been added to your `PATH` environment variable, per the installation methods above.  
Commands will differ slightly if you are invoking the module directly. i.e.) `heteromotility` will be `/path/to/heteromotility.py`.

The general method to calculate motility statistics is:

*Pickled Cell Paths*

    $ heteromotility . output_path/ --exttrack path/to/object_paths.pickle

*Tracks X and Y in CSVs*

    $ heteromotility . output_path/ --tracksX path/to/tracksX.csv --tracksY path/to/tracksY.csv

Both methods will output a CSV named `motility_statistics.csv` in the specified output directory, formatted with features as columns and samples as rows.  
The optional argument `--output_suffix` can be used to add a suffix to the output CSV.

    $ heteromotility path_to_input_csvs/ ./
    $ ls
    motility_statistics.csv
    $ heteromotility path_to_input_csvs/ ./ --output_suffix TEST_SUFFIX
    $ ls
    motility_statistics.csv motility_statistics_TEST_SUFFIX.csv

## Split Motility Path Calculation

Heteromotility supports splitting an object's path into multiple subpaths and calculating features for each subpath. This is useful to investigate changes in an object's motility over time. This feature is triggered with the `--detailedbalance` command line flag, followed by an integer specifying the length of temporal windows to consider. By default, paths are extracted from non-overlapping windows (i.e. the stride is equal to window size). This can be changed by using the `--sliding_window` command line flag followed by an integer specifying the stride of temporal windows for comparison. Additional flags are document in the help menu (`heteromotility -h`) to control the number of windows measured.

**Note:** Subpaths have a minimum length of 20 steps, as multiple `Heteromotility` features rely upon regression analyses that are confounded by exceedingly small path lengths.

This feature is executed like so:

    $ heteromotility /path/to/csvs /output/path/ --detailedbalance $WIDTH --sliding_window $STRIDE

where the `WIDTH` is the size of subpath windows and `STRIDE` is the stride of windows along the time series.

The resulting output files will be named `motility_statistics_split_LENGTH.csv` where `LENGTH` is the subpath size for those features. Importantly, the `cell_ids` column will now contain additional information. In addition to the unique cell identifier, `cell_ids` will contain a dash `-` following the identifier with subpath number, indexed beginning at `0`. The format appears:

    cell_ids ...
    ObjectIdentifier-SubpathNumber ...
    ...

For instance, for a path with total length N = 80, minimum length tau = 20, the `cell_ids` column would appear as follows:

    cell_ids ...
    obj0-0 ...
    obj0-1 ...
    obj0-2 ...
    obj0-3 ...
    obj1-1 ...
    obj1-2 ...
    ...

Where `obj0-0` is the first subpath (length = 20) of `obj0`, `obj0-1` is the second subpath of `obj0`, and so forth.

## Input Data Format

### Tracks X and Y CSVs

Heteromotility accepts object paths as `N x T` CSVs, where `N` is the number of samples and `T` is the number of time steps. One CSV encodes an object's position in the `X` dimension, and another in the `Y` dimension.

Each row describes a single sample, and each column contains the objects position in the `X` or `Y` dimension at the corresponding time step, ordered `0` to `T`, left to right.

For instance, `tracksX.csv` may contain the following:

    0, 5, 4, 5, 7, 5, 10, ...
    38, 51, 42, 38, 41, 43, ...
    ...

and likewise for `tracksY.csv`.

### Pickled Cell Paths

Heteromotility also accepts a pickled Python dictionary of object paths as an input. The dictionary should be keyed by a unique object identifier (i.e. numbers, names), with each key corresponding to a sequential list of XY-point tuples.

    object_paths = {
                    obj1 : [(x1,y1), (x2,y2), (x3,y3)...],
                    obj2 : [(x1,y1), (x2,y2), (x3,y3)...],
                    ...
                    }

To export this dictionary as a pickle, use the standard Python pickle library.

    > import pickle
    > with file('output_name.pickle', 'wb') as of:
    >   pickle.dump(object_paths, of)

# Analysis

## Dimensionality Reduction and Clustering

Functions for dimensionality reduction and clustering are implemented in the `R` language, and may be found in `analysis/hm_common_fnx.R`. An example of a simple analysis script is provided in `mef_analysis.R`.

## Supervised Classification

SVM classification models are implemented in `analysis/python/mef_classif.py`. Functions are also included to perform round-robin analysis of multiple experiments, and a Grid Search for hyperparameter optimization.

## State Dynamics Analysis

State dynamics analyze the output of a series of temporal windows calculated by `Heteromotility`, as outlined above. *N*-dimensional course-grained probability flux analysis (ND-cgPFA) is implemented in `analysis/python/pfa.py`. `pfa.py` also contains tools to quantify statewise transition vectors in PCA space.

Two dimensional cgPFA is also implemented in `R` at `cgpfa.R`. Tools for visualizing divergence in cgPFA state spaces are implemented in `Matlab` in `analysis/pfa_divergence.m`. A visualization run may be performed by quantifying state transitions using `cgpfa.R` and `pfa_divergence.m` to generate both 2D and 3D divergence maps.