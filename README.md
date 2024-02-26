![image](https://github.com/espsrc/meerkat-pipeline/assets/7033451/b0d53933-7902-4ea5-a294-7b8df66822d5)

# The IDIA MeerKAT Pipeline for Custom HPC facilities

The IDIA MeerKAT pipeline is a radio interferometric calibration pipeline designed to process MeerKAT data.


## Requirements

This pipeline is designed to run on the Ilifu cluster @ IDIA, making use of SLURM and MPICASA but with this repository fork **we wanted to enable this pipeline for a customised HPC facility**. In our case this Pipeline has been ported to our HPC computing infrastructure at the espSRC (Spanish SKA Regional Centre @ IAA-CSCI, Granada) within a cluster with Slurm and MPICASA, and the software containers needed.

## Note about data

ℹ️ It is not necessary to copy the raw data (i.e. the MS) to your working directory. The first step of the pipeline does this for you by creating an MMS or MS, and does not attempt to manipulate the raw data (see [data format](https://idia-pipelines.github.io/docs/processMeerKAT/Example-Use-Cases/#data-format)).

## Steps

### 0. Clone this repository

Go to your shared folder in your slurm cluster and type:

        git  clone <this repo url>

then 

        cd meerkat-pipeline


### 1. Setup the pipeline in your environment

In order to use the `processMeerKAT.py` script, source the `setup.sh` file (first check the path where you are):

        pwd

get this path and paste it in `<pathofcode>`:

        source <pathofcode>/setup.sh

which will add the correct paths to your `$PATH` and `$PYTHONPATH` in order to correctly use the pipeline.

### 2. Build a config file:

#### a. For continuum/spectral line processing :

        processMeerKAT.py -B -C myconfig.txt -M mydata.ms

#### b. For polarization processing :

        processMeerKAT.py -B -C myconfig.txt -M mydata.ms -P

#### c. Including self-calibration :

        processMeerKAT.py -B -C myconfig.txt -M mydata.ms -2

#### d. Including science imaging :

        processMeerKAT.py -B -C myconfig.txt -M mydata.ms -I


#### e. Other options and configurations :

This defines several variables that are read by the pipeline while calibrating the data, as well as requesting resources on the cluster. The [config file parameters](https://idia-pipelines.github.io/docs/processMeerKAT/config-files) are described by in-line comments in the config file itself wherever possible. The `[-P --dopol]` option can be used in conjunction with the `[-2 --do2GC]` and `[-I --science_image]` options to enable polarization calibration as well as [self-calibration](https://idia-pipelines.github.io/docs/processMeerKAT/self-calibration-in-processmeerkat) and [science imaging](https://idia-pipelines.github.io/docs/processMeerKAT/science-imaging-in-processmeerkat).

#### Examples :

To create a configuration file for continuum/spectral line processing, self-calibration and science imaging:

        processMeerKAT.py -B -C myconfig.txt -M mydata.ms -2 -I

To create a configuration file  for continuum/spectral line processing just with science imaging: 

        processMeerKAT.py -B -C myconfig.txt -M mydata.ms -I



#### Note on optional configurations
Depending on the option you choose, the configuration file (here, i.e.: ``myconfig.txt``)  will change to include the default settings for Self-Calibration and Imaging.
Here you can see the default settings for each of them:

**Self Calibration:**

```
[selfcal]
nloops = 2                        # Number of clean + bdsf loops.
loop = 0                          # If nonzero, adds this number to nloops to name images or continue previous run
cell = '1.5arcsec'
robust = -0.5
imsize = [6144, 6144]
wprojplanes = 512
niter = [10000, 50000, 50000]
threshold = ['0.5mJy', 10, 10]    # After loop 0, S/N values if >= 1.0, otherwise Jy
nterms = 2                        # Number of taylor terms
gridder = 'wproject'
deconvolver = 'mtmfs'
calmode = ['','p']                # '' to skip solving (will also exclude mask for this loop), 'p' for phase-only and 'ap' for amplitude and phase
solint = ['','1min']
uvrange = ''                      # uv range cutoff for gaincal
flag = True                       # Flag residual column after selfcal?
gaintype = 'G'                    # Use 'T' for polarisation on linear feeds (e.g. MeerKAT)
discard_nloops = 0                # Discard this many selfcal solutions (e.g. from quick and dirty image) during subsequent loops (only considers when calmode !='')
outlier_threshold = 0.0           # S/N values if >= 1.0, otherwise Jy
outlier_radius = 0.0              # Radius in degrees for identifying outliers in RACS
```
More details on self-calibration: https://idia-pipelines.github.io/docs/processMeerKAT/self-calibration-in-processmeerkat/

**Imaging:**

```
[image]
cell = '1.5arcsec'
robust = -0.5
imsize = [6144, 6144]
wprojplanes = 512
niter = 50000
threshold = 10                    # S/N value if >= 1.0 and rmsmap != '', otherwise Jy
multiscale = [0, 5, 10, 15]
nterms = 2                        # Number of taylor terms
gridder = 'wproject'
deconvolver = 'mtmfs'
restoringbeam = ''
stokes = 'I'
pbthreshold = 0.1                 # Threshold below which to mask the PB for PB correction
pbband = 'LBand'                  # Which band to use to generate the PB - one of "LBand" "SBand" or "UHF"
mask = ''
rmsmap = ''
outlierfile = ''
```

More details on Science imaging: https://idia-pipelines.github.io/docs/processMeerKAT/science-imaging-in-processmeerkat/
        
### 3. To run the pipeline

⚠️ Before starting, review the configuration files to match your slurm cluster in terms of number of CPUs, tasks per node and available RAM.

Check ``myconfig.txt``: 

```
...
[slurm]
nodes = 3
ntasks_per_node = 16
plane = 1
mem = 24
partition = 'debug'
... 
```

and ``processMeerKAT/processMeerKAT.py``:

```
...
#Set global limits for current cluster configuration
TOTAL_NODES_LIMIT = 3
CPUS_PER_NODE_LIMIT = 16
NTASKS_PER_NODE_LIMIT = CPUS_PER_NODE_LIMIT
MEM_PER_NODE_GB_LIMIT = 24 
MEM_PER_NODE_GB_LIMIT_HIGHMEM = 26
...
```

Once verified, type the following to create the configuration files according to this configuration: 

        processMeerKAT.py -R -C myconfig.txt

⚠️⚠️⚠️ 
This will not run the pipeline, it will just create the files so you can manually upload them to slurm.
⚠️⚠️⚠️

With this command it will create `submit_pipeline.sh`, which you can then run with `./submit_pipeline.sh` to submit all pipeline jobs to the SLURM queue.

Other convenience scripts are also created that allow you to monitor and (if necessary) kill the jobs.

* `summary.sh` provides a brief overview of the status of the jobs in the pipeline
* `findErrors.sh` checks the log files for commonly reported errors (after the jobs have run)
* `killJobs.sh` kills all the jobs from the current run of the pipeline, ignoring any other (unrelated) jobs you might have running.
* `cleanup.sh` wipes all the intermediate data products created by the pipeline. This is intended to be launched after the pipeline has run and the output is verified to be good.

For more configuration options within the command line, run `processMeerKAT.py -h`, which provides a brief description of all the [command line options](https://idia-pipelines.github.io/docs/processMeerKAT/using-the-pipeline#command-line-options).

## Debugging

For debugging the status of jobs, you can find in the ``logs/`` directory where you have the pipeline, the set of processes that have run or are running.

For example, for each part of the pipeline you will see (templates are ```<operation>_<job_id>.<output>```):

**Partition:**
```
partition-3105_0.err
partition-3105_0.out
partition-3105_1.err
partition-3105_1.out
partition-3105_10.err
partition-3105_10.out
partition-3105_2.err
partition-3105_2.out
partition-3105_3.err
partition-3105_3.out
partition-3105_4.err
partition-3105_4.out
partition-3105_5.err
partition-3105_5.out
partition-3105_6.err
partition-3105_6.out
partition-3105_7.err
partition-3105_7.out
partition-3105_8.err
partition-3105_8.out
partition-3105_9.err
partition-3105_9.out
```

**Concat:**

```
concat-3217.casa
concat-3217.err
concat-3217.mpi
concat-3217.out
```

**PlotCalibration SPW:**

```
plotcal_spw-3218.casa
plotcal_spw-3218.err
plotcal_spw-3218.mpi
plotcal_spw-3218.out
```

**SelfCalibration:**
```
selfcal_part1-3230.casa
selfcal_part1-3230.err
selfcal_part1-3230.out
selfcal_part2-3231.casa
selfcal_part2-3231.err
selfcal_part2-3231.out
```

**Science Image:**

```
science_image-3232.out
science_image-3232.casa
science_image-3232.err
```




## Using multiple spectral windows (new in v1.1)

Starting with v1.1 of the processMeerKAT pipeline, the default behaviour is to split up the MeerKAT band into several spectral windows (SPWs), and process each concurrently. This results in a few major usability changes as outlined below:

1. **Calibration output** : Since the calibration is performed independently per SPW, all the output specific to that SPW is within its own directory. Output such as the calibration tables, logs, plots etc. per SPW can be found within each SPW directory.

2. **Logs in the top level directory** : Logs in the top level directory (*i.e.,* the directory where the pipeline was launched) correspond to the scripts in the `precal_scripts` and `postcal_scripts` variables in the config file. These scripts are run from the top level before and after calibration respectively. By default these correspond to the scripts to calculate the reference antenna (if enabled), partition the data into SPWs, and concat the individual SPWs back into a single MS/MMS.

More detailed information about SPW splitting is found [here](https://idia-pipelines.github.io/docs/processMeerKAT/config-files#spw-splitting).

The documentation can be accessed on the [pipelines website](https://idia-pipelines.github.io/docs/processMeerKAT), or on the [Github wiki](https://github.com/idia-astro/pipelines/wiki).
