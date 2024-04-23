# Installation

DNBelab_C_Series_HT_singlecell-analysis-software analysis software can be based on conda environment or docker/singularity container.

### 1. Conda

##### Create dnbc4tools environment

1.1 Requires [conda](https://docs.anaconda.com/free/miniconda/) to be installed

```shell
source /miniconda3/bin/activate
conda create -n dnbc4tools python=3.8 perl
conda activate dnbc4tools
```
1.2 Install dnbc4tools

```shell
conda install -c conda-forge -c bioconda htslib=1.18 samtools=1.18
pip install dnbc4tools==2.1.2
```
Successfully installed dnbc4tools

</br>

Notes:
- If the `annoy` package installation fails,  use `conda install conda-forge::python-annoy`.
- You can also create dnbc4tools environment using YAML:
```
wget https://raw.githubusercontent.com/MGI-tech-bioinformatics/DNBelab_C_Series_HT_scRNA-analysis-software/version2.0/dnbc4tools.yaml
conda env create -f dnbc4tools.yaml -n dnbc4tools
```

### 2. container

##### Singularity

```shell
singularity build dnbc4tools.sif docker://dnbelabc4/dnbc4tools
```
##### Docker

```shell
docker pull dnbelabc4/dnbc4tools
```
