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

This repo contains a standalone Python script ([`dereplicator.py`](dereplicator.py)) to solve a problem we ran into while analysing a large public genome dataset for carbapenemases. I occasionally run into: dereplicating a group of bacterial genome assemblies. Dereplication means removing assemblies for which there are sufficiently close relatives, resulting in a smaller set where the assemblies are more unique.
I adapted the style of the script from Ryan Wick's Assembly-Dereplicator (https://github.com/rrwick/Assembly-Dereplicator) and tried to make it work very similar to his script. While his script compares assemblies and finally outputs representative genomes into an output folder, this Python script takes in as input, genome assemblies in a folder, runs an All vs All mash pariwise distance estimation across all genomes and applies a distance threshold to group the assemblies into clusters and outputs the results of genomes and the cluster they belong to as a tab separated text file.

This is to solve the problem of identifying potential outbreaks of a particular bacterial strain (ST) across different countries. 
As an example, imagine you have 10000 genome assemblies for a particular taxon and want to do some analysis on them, maybe building a pan genome. You know there is redundancy in this set because some of the genomes come from outbreaks and are nearly identical to each other. So instead of doing the analysis on all 10000 assemblies, you can dereplicate them to a smaller set (i.e. remove near-identical redundant genomes) so your analysis will be faster.


## Example

<p align="center"><picture><source srcset="images/trees-dark.png" media="(prefers-color-scheme: dark)"><img src="images/trees.png" alt="Verticall trees" width="90%"></picture></p>

To give you a visual idea of how this works, here are trees built from 1000 assemblies from the genus _Klebsiella_ dereplicated to various distance levels. You can see that in the no-dereplication and lower-distance trees, one clade dominates (corresponding to _K. pneumoniae_, the most sequenced species in the genus). The higher-distance trees are more balanced, and when dereplicated to a distance of 0.025, there is only one assembly per species/subspecies.


## Requirements

I initially tried to use all libraries that are usually included in standard python libraries but in order to speed up the process of mash comparisons, then clustering based on distances and output as well as displaying a progress bar, I had to use some additional libraries.
To make it easier to use, I have included a conda environment file (assemblycluster.yml), which you can use for creating an environment to run the script in. I have also included Mash itself in it - so it's essentially plug and play.


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
git clone https://github.com/rrwick/Assembly-dereplicator
cp Assembly-dereplicator/dereplicator.py ~/.local/bin
dereplicator.py --help
```

If you'd like to double-check that everything works as intended, you can run this repo's [automated tests](test).



## Method

The basic dereplication process is to find the closest pair of assemblies in the set, discard the assembly in that pair with the lower N50 value and repeat until a stop condition is reached. By using N50 as a metric of assembly quality, complete assemblies are preferentially kept over draft assemblies.

There are three possible ways to specify how much to dereplicate:
* A minimum pairwise distance, e.g. `--distance 0.001`. This will make dereplication continue until no two assemblies in the set have a Mash distance of less than the given value.
* An assembly count, e.g. `--count 100`.  This will make dereplication continue until the target number of assemblies is reached.
* An assembly fraction, e.g. `--fraction 0.5`.  This will make dereplication continue until the target fraction of assemblies is reached.

If multiple criteria are used, then dereplication will continue until all are satisfied. E.g. `--distance 0.001 --count 100` will ensure that no two assemblies have a Mash distance of greater than 0.001 and there are no more than 100 assemblies.

The process is deterministic, so running the script multiple times will give the same result. Also, running the script in multiple steps (e.g. first with `--distance 0.01` then again with `--distance 0.02`) will give the same result as running in a single step (e.g. using `--distance 0.02` the first time).



## Quick usage

Assembly Dereplicator is easy to run. Just give it an input directory containing your assemblies in FASTA format, an output directory and a value for `--distance`, `--count` or `--fraction`:
```bash
dereplicator.py --distance 0.01 assembly_dir output_dir
```



## Full usage

```
usage: dereplicator.py [--distance DISTANCE] [--count COUNT] [--fraction FRACTION]
                       [--sketch_size SKETCH_SIZE] [--threads THREADS] [--verbose] [-h]
                       [--version]
                       in_dir out_dir

Assembly Dereplicator

Positional arguments:
  in_dir                     Directory containing all assemblies
  out_dir                    Directory where dereplicated assemblies will be copied

Dereplication target:
  --distance DISTANCE        Dereplicate until the closest pair has a Mash distance of this value
                             or greater
  --count COUNT              Dereplicate until there are no more than this many assemblies
  --fraction FRACTION        Dereplicate until there are no more than this fraction of the
                             original number of assemblies

Settings:
  --sketch_size SKETCH_SIZE  Mash assembly sketch size (default: 10000)
  --threads THREADS          Number of CPU threads for Mash (default: 10)
  --verbose                  Display more output information

Other:
  -h, --help                 Show this help message and exit
  --version                  Show program's version number and exit
```

Positional arguments:
* `in_dir`: directory containing input assemblies, will be searched recursively. The input assemblies must be in FASTA format and the script will find them in the input directory based on extension. Any of the following are acceptable: `.fasta`, `.fasta.gz`, `.fna`, `.fna.gz`, `.fa` and `.fa.gz`.
* `out_dir`: directory where dereplicated assemblies (i.e. the representative assembly for each cluster) will be copied. If this directory doesn't exist, it will be created. Copied assemblies will have the same filename they had in the input directory.

Dereplication target:
* `--distance`: the target minimum pairwise Mash distance between assemblies. Must be between 0 and 1. Mash distance roughly corresponds to one minus average nucleotide identity. Setting it to a small value (e.g. 0.001) will result in more dereplicated assemblies, i.e. only very close relatives will be excluded. Setting it to a large value (e.g. 0.02) will result in fewer dereplicated assemblies, perhaps just one per species.
* `--count`: the target number of output assemblies. If this value is equal to or greater than the number of input assemblies, no dereplication will take place.
* `--fraction`: the target fraction of output assemblies. Must be between 0 and 1.
* At least one of `--distance`, `--count` or `--fraction` must be used. Multiple targets can be used (e.g. both `--distance` and `--count`), in which case dereplication will continue until all targets are met.

Settings:
* `--sketch_size`: controls the [Mash sketch size](https://mash.readthedocs.io/en/latest/sketches.html#sketch-size). A smaller value (e.g. 1000) will make the process faster but slightly less accurate.
* `--threads`: controls how many threads Mash uses for its sketching and distances. Mash scales well in parallel, so use lots of threads if you have them!
* `--verbose`: using this flag results in more information being printed to stdout.



## Performance

Since `dereplicator.py` uses pairwise distances, it scales with the square of the number of input assemblies, i.e. _O_(_n_<sup>2</sup>). This means that small numbers of input assemblies (e.g. <100) will be very fast, and large numbers (e.g. >10000) will take much longer. Mash parallelises well, so use as many threads as you can for large datasets.

As a test, I ran `dereplicator.py` on all [_Klebsiella_ genomes in GTDB](https://gtdb.ecogenomic.org/tree?r=g__Klebsiella). At the time of writing, this was about 13700 assemblies, so about 188 million pairwise comparisons. Using 48 CPU threads, the process took about 40 minutes to complete and used about 35 GB of RAM.



## License

[GNU General Public License, version 3](https://www.gnu.org/licenses/gpl-3.0.html)
