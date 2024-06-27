[![Github Release](https://img.shields.io/github/v/release/MGI-tech-bioinformatics/DNBelab_C_Series_HT_scRNA-analysis-software)](https://github.com/MGI-tech-bioinformatics/DNBelab_C_Series_HT_scRNA-analysis-software/releases)
[![PyPI](https://img.shields.io/pypi/v/dnbc4tools)](https://pypi.org/project/DNBC4tools)
[![Docker Pulls](https://img.shields.io/docker/pulls/dnbelabc4/dnbc4tools)](https://hub.docker.com/r/dnbelabc4/dnbc4tools)

# DNBelab_C_Series_HT_singlecell-analysis-software

## Introduction

An open source and flexible pipeline to analyze high-throughput DNBelab C Series<sup>TM</sup> single-cell datasets. 

**Hardware/Software requirements** 

- x86-64 compatible processors.
- require at least 50GB of RAM and 4 CPU. 
- centos 7.x 64-bit operating system (Linux kernel 3.10.0, compatible with higher software and hardware configuration). 

## Start

- [**installation** ](./doc/installation.md)
- [**quick start** ](./doc/quickstart.md)

## Support

- Please use github issue tracker for questions. [**issues**](https://github.com/MGI-tech-bioinformatics/DNBelab_C_Series_HT_scRNA-analysis-software/issues)


> [!TIP]
>
> CHANGELOG:  2.1.3 pre-release
>
> 1. Added single-cell immune repertoire analysis.
> 2. Optimizing high memory usage when performing combined single-cell ATAC analysis
> 3. Improved single-cell RNA sequencing I/O for high-thread scenarios, reducing analysis time.
> 4. Added the function of checking whether the gtf format is correct and generating a new gtf file in the correct format.
