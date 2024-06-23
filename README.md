# Assembly Cluster

## Table of contents

* [Introduction](#introduction)
* [Requirements](#requirements)
* [Installation](#installation)
* [Method](#method)
* [Quick usage](#quick-usage)
* [Full usage](#full-usage)
* [License](#license)



## Introduction

This repo contains a standalone Python script ([`assemblycluster.py`](assemblycluster.py)) to solve a problem we ran into while analysing a large public genome dataset for carbapenemases. I occasionally run into: dereplicating a group of bacterial genome assemblies. Dereplication means removing assemblies for which there are sufficiently close relatives, resulting in a smaller set where the assemblies are more unique.

I adapted the style of the script from Ryan Wick's Assembly-Dereplicator (https://github.com/rrwick/Assembly-Dereplicator) and tried to make it work very similar to his script. While his script compares assemblies and finally outputs representative genomes into an output folder, this Python script takes in as input, genome assemblies in a folder, runs an All vs All mash pariwise distance estimation across all genomes and applies a distance threshold to group the assemblies into clusters and outputs the results of genomes and the cluster they belong to as a tab separated text file.

This is to solve the problem of identifying potential outbreaks of a particular bacterial strain (ST) across different countries.


## Requirements

I initially tried to use all libraries that are usually included in standard python libraries but in order to speed up the process of mash comparisons, then clustering based on distances and output as well as displaying a progress bar, I had to use some additional libraries.
To make it easier to use, I have included a conda environment file (`assemblycluster.yml`), which you can use for creating an environment to run the script in. I have also included Mash itself in it - so it's essentially plug and play.


## Installation

Since it is a single script, no installation is required. You can simply clone it, create conda environment using yml file, activate it and run:
```bash
git clone https://github.com/bhargava-morampalli/Assembly-Cluster
mamba env create -f assemblycluster.yml
mamba activate assemblycluster
Assembly-Cluster/assemblycluster.py --help
```

If you plan on using it often, you can copy it to someplace in your PATH variable for easier access:
```bash
git clone https://github.com/bhargava-morampalli/Assembly-Cluster
cp Assembly-Cluster/assemblycluster.py /usr/local/bin
assemblycluster.py --help
```

## Method

The basic clustering process is to running an All vs All Mash pariwise distance estimation to calculate distances between all the assemblies. Then applying a distance threshold (default - `0.001`), assemblies that have distance estimation of less then 0.001 amongst all of them are grouped together. This means that only very closely related assemblies are grouped together when using the default distance threshold.


## Quick usage

Assembly Cluster is easy to run. Just give it an input directory containing your assemblies in FASTA format or if it needs to be run for multiple sets of fasta files separately - a parent directory containing all the subdirectories that need to be processed, a value for `--threshold`, `--threads` as needed for specific cases.

```bash
assemblycluster.py input_dir
assemblycluster.py input_dir --threshold 0.02 --threads 32
```



## Full usage

```
usage: assemblycluster.py [-h] [--threshold THRESHOLD] [--threads THREADS] input_dir

Group assemblies based on Mash distances

positional arguments:
  input_dir             Directory containing assembly files or subdirectories

optional arguments:
  -h, --help            show this help message and exit
  --threshold THRESHOLD
                        Mash distance threshold for grouping (default: 0.001)
  --threads THREADS     Number of threads for parallel processing (default: 10)

Examples:
  Process a single folder of assemblies:
    python script.py /path/to/assembly/folder

  Process a folder with multiple subdirectories containing assemblies:
    python script.py /path/to/parent/folder

  Specify a custom threshold and number of threads:
    python script.py /path/to/folder --threshold 0.005 --threads 16

Notes:
  - The script automatically detects whether it's processing a single folder
    or multiple subdirectories.
  - Output files are named '<foldername>_grouped.txt' and placed in the
    respective folders.
  - The script scales well with increased threads, but performance gains may
    plateau depending on I/O limitations and the number of CPU cores available.
```

Positional arguments:
* `input_dir`: directory containing input assemblies or a parent directory that contains multiple subdirectories with a set of fasta files in each that needs se. The input assemblies must be in FASTA format and the script will find them in the input directory based on extension. Any of the following are acceptable: `.fasta`, `.fasta.gz`, `.fna`, `.fna.gz`, `.fa` and `.fa.gz`.
* Output text file is always created in the `input_dir` which was supplied. If a parent directory is supplied, output text file is always created in the respective subfolders.

Cluster distance threshold:
* `--threshold`: the target minimum pairwise Mash distance between assemblies. Must be between 0 and 1. Default value is 0.001. Mash distance roughly corresponds to one minus average nucleotide identity. Setting it to a small value (e.g. 0.001) will result in large number of clusters, i.e. only very close relatives are grouped together. Setting it to a large value (e.g. 0.02) will result in fewer clusters.
* Default value of 0.001 is assumed if no value is given as `--threshold`. If a different value is needed, `--threshold VALUE` needs to be given.

Settings:
* `--threads`: controls how many threads Mash uses for its sketching and distances. Default is set to 10 (no need to mention it). If a different valye is needed, `--threads VALUE` needs to be given. All vs All Mash comparisons need lot of resources depending on the number of genomes/assemblies - so, use as many as you can spare.

## License

[GNU General Public License, version 3](https://www.gnu.org/licenses/gpl-3.0.html)
