# Installation

DNBelab_C_Series_HT_singlecell-analysis-software analysis software can be based on conda environment or docker/singularity container.

### 1. Conda

##### 1.1 Clone repo

```shell
git clone -b DNBC4_Dev https://github.com/lishuangshuang0616/DNBelab_C_Series_HT_scRNA-analysis-software.git
chmod 755 -R DNBelab_C_Series_HT_scRNA-analysis-software
cd DNBelab_C_Series_HT_scRNA-analysis-software
```

##### 1.2 Create dnbc4tools environment

Requires conda to be installed

```shell
source /miniconda3/bin/activate
conda env create -f dnbc4dev.yaml -n dnbc4dev
```

### 2. container

##### Singularity

```shell
singularity build dnbc4tools.sif docker://dnbelabc4/dnbc4dev
```
##### Docker

```shell
docker pull dnbelabc4/dnbc4dev
```
