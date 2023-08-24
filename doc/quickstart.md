# Quick start

### 1. dnbc4tools

- **conda**

No source environment is required, use the full path command directly

```shell
$miniconda3/envs/dnbc4dev/bin/dnbc4tools
```

- **docker**

```shell
docker run -P  -v $Database_LOCAL:/database -v $Rawdata_LOCAL:/data -v $Result_LOCAL:/result dnbelabc4/dnbc4dev dnbc4tools
# $Database_LOCAL: directory on your local machine that has the database files. 
# $Rawdata_LOCAL: directory on your local machine that has the sequence data.
# $Result_LOCAL: directory for result.
```

- **singularity**

```shell
export SINGULARITY_BIND=$data,$result,$database
singularity exec dnbc4dev.sif dnbc4tools
# You can bind multiple directories by Using the environment variable:
# export SINGULARITY_BINDPATH=$data,$result,$database
```



### 2. scRNA

#### 2.1 Build index for reference genome

- **Human(GRCh38)**

```shell
wget http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_32/GRCh38.primary_assembly.genome.fa.gz
wget http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_32/gencode.v32.primary_assembly.annotation.gtf.gz
gzip -d GRCh38.primary_assembly.genome.fa.gz
gzip -d gencode.v32.primary_assembly.annotation.gtf.gz

$dnbc4tools tools mkgtf --ingtf gencode.v32.primary_assembly.annotation.gtf --output genes.filter.gtf --type gene_type
                        
$dnbc4tools rna mkref --ingtf genes.filter.gtf --fasta GRCh38.primary_assembly.genome.fa --threads 10 --species Homo_sapiens
```

- **Mouse(GRCm38)**

```shell
wget http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M23/GRCm38.primary_assembly.genome.fa.gz
wget http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M23/gencode.vM23.primary_assembly.annotation.gtf.gz
gzip -d GRCm38.primary_assembly.genome.fa.gz
gzip -d gencode.vM23.primary_assembly.annotation.gtf.gz

$dnbc4tools tools mkgtf --ingtf gencode.vM23.primary_assembly.annotation.gtf --output genes.filter.gtf --type gene_type
                        
$dnbc4tools rna mkref --ingtf genes.filter.gtf --fasta GRCm38.primary_assembly.genome.fa --threads 10 --species Mus_musculus
```

> *Note: version 2.0 has built the reference database using the parameter "--noindex" to skip the scStar index generation process*

#### 2.2 RUN

**Running the entire workflow**

```shell
$dnbc4tools rna run \
	--cDNAfastq1 /test/data/test_cDNA_R1.fastq.gz \
	--cDNAfastq2 /test/data/test_cDNA_R2.fastq.gz \
	--oligofastq1 /test/data/test_oligo1_1.fq.gz,/test/data/test_oligo2_1.fq.gz \
	--oligofastq2 /test/data/test_oligo1_2.fq.gz,/test/data/test_oligo2_2.fq.gz \
	--genomeDir /database/scRNA/Mus_musculus/mm10  \
	--name test --threads 10
```

**Use the multi command to process multiple samples**

```shell
$dnbc4tools rna multi --list samplelist \
         --genomeDir /database/scRNA/Mus_musculus/mm10 \
         --threads 10
```



### 3. scATAC

#### 3.1 Build index for reference genome

- **Human(GRCh38)**

```shell
wget http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_32/GRCh38.primary_assembly.genome.fa.gz
wget http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_32/gencode.v32.primary_assembly.annotation.gtf.gz
gzip -d GRCh38.primary_assembly.genome.fa.gz
gzip -d gencode.v32.primary_assembly.annotation.gtf.gz

$dnbc4tools tools mkgtf --ingtf gencode.v32.primary_assembly.annotation.gtf --output genes.filter.gtf --type gene_type
                        
$dnbc4tools atac mkref --fasta GRCh38.primary_assembly.genome.fa --ingtf genes.filter.gtf --species Homo_sapiens --blacklist hg38 --prefix chr
```

- **Mouse(GRCm38)**

```shell
wget http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M23/GRCm38.primary_assembly.genome.fa.gz
wget http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M23/gencode.vM23.primary_assembly.annotation.gtf.gz
gzip -d GRCm38.primary_assembly.genome.fa.gz
gzip -d gencode.vM23.primary_assembly.annotation.gtf.gz

$dnbc4tools tools mkgtf --ingtf gencode.vM23.primary_assembly.annotation.gtf --output genes.filter.gtf --type gene_type
                        
$dnbc4tools atac mkref --fasta GRCm38.primary_assembly.genome.fa --ingtf genes.filter.gtf --species Mus_musculus --blacklist mm10 --prefix chr
```

#### 3.2 RUN

**Running the entire workflow**

```shell
$dnbc4tools atac run \
	--fastq1 /test/data/test1_R1.fastq.gz,/test/data/test2_R1.fastq.gz \
	--fastq2 /test/data/test1_R2.fastq.gz,/test/data/test2_R2.fastq.gz \
	--genomeDir /database/scATAC/Mus_musculus/mm10  \
	--name test --threads 10
```

**Use the multi command to process multiple samples**

```shell
$dnbc4tools atac multi --list samplelist \
         --genomeDir /database/scATAC/Mus_musculus/mm10  \
         --threads 10
```

