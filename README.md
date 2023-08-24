[![PyPI](https://img.shields.io/pypi/v/dnbc4dev)](https://pypi.org/project/DNBC4dev)
[![Docker Pulls](https://img.shields.io/docker/pulls/dnbelabc4/dnbc4dev)](https://hub.docker.com/r/dnbelabc4/dnbc4dev)

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
- Note: *Upgrading to version dev will necessitate recreating the analysis environment. If you downloaded the previous dev version, you only need to manually install the python package **pyahocorasick** and update **dnbc4dev***.
- Note: *Upgrading to version dev requires rebuilding the scRNA reference database. For more details, please refer to the quick start guide.If you have processed the database according to the dev version or 2.1.0 version, you do not need to re-modify.*
