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
pip install dnbc4tools==2.1.3rc1
```
Successfully installed dnbc4tools

</br>

>  [!NOTE]
>
> - If the `annoy` package installation fails,  use `conda install conda-forge::python-annoy`.
> - **Update**: no need to reinstall the environment. Use `pip install dnbc4tools==[version]` to install the specified version.



### 2. container

##### Singularity

```shell
singularity build dnbc4tools.sif docker://dnbelabc4/dnbc4tools:2.1.3rc1
```
##### Docker

```shell
docker pull dnbelabc4/dnbc4tools:2.1.3rc1
```
