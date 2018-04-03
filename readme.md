# PoolSex

## Overview

The PoolSex pipeline is used to analyze pooled sequencing data with focus on sex. This specific component of the pipeline is used to generate shell files for each part of the pipeline and submit them on an SGE scheduler.

## Requirements

- Python (>= 3.5)
- Bwa
- Samtools
- Picard tools
- Popoolation

## Installation

- Clone: `git clone git@github.com:INRA-LPGP/PoolSex.git`
- Alternative: Download the archive and unzip it

## Quickstart

- Create a basic input directory. This directory should have the following structure:

```.
├─── genomes
|     ├────── <species_name>_genome.<fasta/fa/fna>
├─── reads
|     ├────── <sex>_<lane>_<mate_number>.<fasta/fastq><.gz>
|     ├────── <sex>_<lane>_<mate_number>.<fasta/fastq><.gz>
|     └────── ...
└─── settings.txt
```

- The file `settings.txt` contains important settings values. Each value is specified on one line with the syntax `setting=value`. For instance, the number of threads to use can be set to 16 with the line `threads=16`.

The following settings are available:

Setting          | Description                                       | Default value
---------------- | ------------------------------------------------- | -----------------
threads          | Number of threads to use when possible            | `16`
mem              | Total memory to allocate                          | `21G`
bwa              | Path to BWA binary                                | `bwa`
samtools         | Path to Samtools binary                           | `samtools`
popoolation      | Path to Popoolation mpileup2sync.jar              | `mpileup2sync.jar`
picard           | Path to Picard java binary                        | `picard.jar`
java             | Path to java JRE                                  | `java`
java_mem         | Total memory allocated to java                    | `20G`
max_file_handles | Maximum of file handles for Picard MarkDuplicates | `1000`
java_temp_dir    | Path to java temporary files folder               | `results/temp`

- Run `python3 poolsex.py init -i path_to_folder --run-jobs`

- The final output files will be located in the `results` folder under the name(s) `mpileup2sync_<sex1>_<sex2>.sync`.

- These files can then be used as input for the `poolsex_analysis` software

## Description

The PoolSex pipeline generates and runs scripts to process pooled sequencing data for the analysis of sex determination. This processing is divided in 8 steps:

- Indexing the reference genome with BWA
- Mapping the reads to the reference genome with BWA
- Sorting the resulting BAM files with Samtools
- Adding read groups to the BAM files with Picard AddReadGroups (to improve removal of PCR duplicates)
- Merging BAM files for each sex when sequencing was performed on multiple lanes
- Removing PCR duplicates with Picard MarkDuplicates
- Generating a pileup file with Samtools
- Generating a sync file with Popoolation

The pipeline is designed to run on a computational platform using an SGE scheduler.

The resulting sync file is used as the main input in the `poolsex_analysis` software (https://github.com/INRA-LPGP/poolsex_analysis).

## Usage

### General

`python3 poolsex.py <command> [options]`

**Available commands** :

Command            | Description
------------------ | ------------
`init`             | Generate a shell script to initiate the pipeline from an input directory
`clean`            | Cleanup all files generated by this pipeline in a director
`restart`          | Restart the pipeline from the last completed step or from a chosen step

### init

`python3 poolsex.py init -i input_dir_path [ --run-jobs ]`

*Generate shell files for every step of the pipeline as well as a global shell file to run the pipeline, and can submit the jobs to a SGE scheduler.*

**Options** :

Option | Long flag | Description
--- | --- | ---
`-i` | --input-folder | Path to a basic input directory with the correct structure |
`-r` | --run-jobs | Submit the jobs to a SGE scheduler (qsub) |
`-c` | --clean-temp | Delete results from intermediate steps. Only index files, BAM files are duplicates removal, and mpileup2sync results will be kept. |

**Input directory structure** :

```
.
├─── genomes
|     ├────── <species_name>_genome.<fasta/fa/fna>
├─── reads
|     ├────── <sex>_<lane>_<mate_number>.<fasta/fastq><.gz>
|     ├────── <sex>_<lane>_<mate_number>.<fasta/fastq><.gz>
|     └────── ...
└─── settings.txt
```

**Settings file** :

The settings file is used to define the values of the pipeline's parameters, which are summarized in the following table:

Setting          | Description                                       | Default value
---------------- | ------------------------------------------------- | -----------------
threads          | Number of threads to use when possible            | `16`
mem              | Total memory to allocate                          | `21G`
bwa              | Path to BWA binary                                | `bwa`
samtools         | Path to Samtools binary                           | `samtools`
popoolation      | Path to Popoolation mpileup2sync.jar              | `mpileup2sync.jar`
picard           | Path to Picard java binary                        | `picard.jar`
java             | Path to java JRE                                  | `java`
java_mem         | Total memory allocated to java                    | `20G`
max_file_handles | Maximum of file handles for Picard MarkDuplicates | `1000`
java_temp_dir    | Path to java temporary files folder               | `results/temp`

Each value is specified on one line with the syntax `setting=value`. Default values in the previous table are used when no user-specified value is available. Below is an example of *settings.txt* file to set the number of threads to **8** and the total memory to **45G**:
```
threads=16
mem=45G
```

### clean

`python3 poolsex.py clean -i input_dir_path`

*Clean shell files, results files, and qsub output files from a PoolSex directory*

**Options** :

Option | Long flag | Description
--- | --- | ---
`-i` | --input-folder | Path to a PoolSex input-folder |

### restart

`python3 poolsex.py restart -i input_dir_path [ -s step --run-jobs ]`

*Restart the pipeline from the last completed step or from a specified step*

**Options** :

Option | Long flag | Description
--- | --- | ---
`-i` | --input-folder | Path to a PoolSex input-folder |
`-s` | --step | Step to restart from (index, mapping, sort, groups, merge, duplicates, mpileup, mpileup2sync) |
`-r` | --run-jobs | Submit the jobs to a SGE scheduler (qsub) |
`-c` | --clean-temp | Delete results from intermediate steps. Only index files, BAM files are duplicates removal, and mpileup2sync results will be kept. |

## LICENSE

Copyright (C) 2018 Romain Feron and INRA LPGP

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <https://www.gnu.org/licenses/>
