# Installation

DNBelab_C_Series_HT_singlecell-analysis-software analysis software can be based on conda environment or docker/singularity container.

### 1. Conda

##### 1.1 Clone repo

```shell
git clone https://github.com/MGI-tech-bioinformatics/DNBelab_C_Series_HT_scRNA-analysis-software.git
chmod 755 -R DNBelab_C_Series_HT_scRNA-analysis-software
cd DNBelab_C_Series_HT_scRNA-analysis-software
```

##### 1.2 Create dnbc4tools environment

Requires conda to be installed

```shell
source /miniconda3/bin/activate
conda env create -f dnbc4tools.yaml -n dnbc4tools
```
> *Note: Updated the environment from version 2.1.0 to the new version without the need for a fresh installation. You only need to install the **'pyahocorasick'** package within the dnbc4tools conda environment and update **'dnbc4tools'** as follows:*

### 2. container

##### Singularity

```shell
singularity build dnbc4tools.sif docker://dnbelabc4/dnbc4tools
```
##### Docker

```shell
docker pull dnbelabc4/dnbc4tools
```
