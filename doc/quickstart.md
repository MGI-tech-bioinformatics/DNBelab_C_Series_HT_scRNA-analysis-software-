# Quick Start

> [!Tip]
>
> - ```$dnbc4tools```refers to the path of the executable program. Before running any commands, replace it with the actual installation path. For example, if installed in ```/opt/software/dnbc4tools2.1.3```, use:
>   ```shell
>   /opt/software/dnbc4tools2.1.3/dnbc4tools rna run ...
>   ```
> - Use the line continuation character `\` to split long commands into multiple lines for readability. If the command is on a single line, omit the backslashes.

</br>
</br>

## 1. Single-Cell RNA

### 1.1 Building the Reference Genome

- **Human (GRCh38)**

  ```shell
  wget http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_32/GRCh38.primary_assembly.genome.fa.gz
  wget http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_32/gencode.v32.primary_assembly.annotation.gtf.gz
  gzip -d GRCh38.primary_assembly.genome.fa.gz
  gzip -d gencode.v32.primary_assembly.annotation.gtf.gz
  
  $dnbc4tools tools mkgtf --ingtf gencode.v32.primary_assembly.annotation.gtf --output genes.filter.gtf --type gene_type
  $dnbc4tools rna mkref --ingtf genes.filter.gtf --fasta GRCh38.primary_assembly.genome.fa --threads 10 --species Homo_sapiens
  ```

- **Mouse (GRCm38)**

  ```shell
  wget http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M23/GRCm38.primary_assembly.genome.fa.gz
  wget http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M23/gencode.vM23.primary_assembly.annotation.gtf.gz
  gzip -d GRCm38.primary_assembly.genome.fa.gz
  gzip -d gencode.vM23.primary_assembly.annotation.gtf.gz
  
  $dnbc4tools tools mkgtf --ingtf gencode.vM23.primary_assembly.annotation.gtf --output genes.filter.gtf --type gene_type
  $dnbc4tools rna mkref --ingtf genes.filter.gtf --fasta GRCm38.primary_assembly.genome.fa --threads 10 --species Mus_musculus
  ```

</br>

### 1.2 Data Analysis

```shell
$dnbc4tools rna run \
    --cDNAfastq1 /test/data/test_cDNA_R1.fastq.gz \
    --cDNAfastq2 /test/data/test_cDNA_R2.fastq.gz \
    --oligofastq1 /test/data/test_oligo1_1.fq.gz,/test/data/test_oligo2_1.fq.gz \
    --oligofastq2 /test/data/test_oligo1_2.fq.gz,/test/data/test_oligo2_2.fq.gz \
    --genomeDir /database/scRNA/Mus_musculus/mm10 \
    --name test --threads 10
```

</br>
</br>

## 2. Single-Cell ATAC

### 2.1 Building the Reference Genome

- **Human (GRCh38)**

  ```shell
  wget http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_32/GRCh38.primary_assembly.genome.fa.gz
  wget http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_32/gencode.v32.primary_assembly.annotation.gtf.gz
  gzip -d GRCh38.primary_assembly.genome.fa.gz
  gzip -d gencode.v32.primary_assembly.annotation.gtf.gz
  
  $dnbc4tools tools mkgtf --ingtf gencode.v32.primary_assembly.annotation.gtf --output genes.filter.gtf --type gene_type
  $dnbc4tools atac mkref --fasta GRCh38.primary_assembly.genome.fa --ingtf genes.filter.gtf --species Homo_sapiens --prefix chr
  ```

- **Mouse (GRCm38)**

  ```shell
  wget http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M23/GRCm38.primary_assembly.genome.fa.gz
  wget http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M23/gencode.vM23.primary_assembly.annotation.gtf.gz
  gzip -d GRCm38.primary_assembly.genome.fa.gz
  gzip -d gencode.vM23.primary_assembly.annotation.gtf.gz
  
  $dnbc4tools tools mkgtf --ingtf gencode.vM23.primary_assembly.annotation.gtf --output genes.filter.gtf --type gene_type
  $dnbc4tools atac mkref --fasta GRCm38.primary_assembly.genome.fa --ingtf genes.filter.gtf --species Mus_musculus --prefix chr
  ```

</br>

### 2.2 Data Analysis

```shell
$dnbc4tools atac run \
    --fastq1 /test/data/test1_R1.fastq.gz,/test/data/test2_R1.fastq.gz \
    --fastq2 /test/data/test1_R2.fastq.gz,/test/data/test2_R2.fastq.gz \
    --genomeDir /database/scATAC/Mus_musculus/mm10 \
    --name test --threads 10
```

</br>
</br>

## 3. Single-Cell VDJ

The single-cell VDJ analysis has pre-built databases for human and mouse and requires no additional reference construction.

### 3.1 Data Analysis

#### 5' scRNA Analysis

```shell
$dnbc4tools rna run \
    --cDNAfastq1 /test/data/test_cDNA_R1.fastq.gz \
    --cDNAfastq2 /test/data/test_cDNA_R2.fastq.gz \
    --oligofastq1 /test/data/test_oligo1_1.fq.gz,/test/data/test_oligo2_1.fq.gz \
    --oligofastq2 /test/data/test_oligo1_2.fq.gz,/test/data/test_oligo2_2.fq.gz \
    --genomeDir /database/scRNA/Homo_sapiens \
    --name test \
    --threads 10 \
    --end5
```

</br>

#### TCR Data Analysis

```shell
$dnbc4tools vdj run \
    --fastq1 /test/data/test1_R1.fastq.gz,/test/data/test2_R1.fastq.gz \
    --fastq2 /test/data/test1_R2.fastq.gz,/test/data/test2_R2.fastq.gz \
    --beadstrans /scRNA/test/output/singlecell.csv \
    --ref Human \
    --name test_tcr \
    --threads 10 \
    --chain TR
```

</br>

#### BCR Data Analysis

```shell
$dnbc4tools vdj run \
    --fastq1 /test/data/test3_R1.fastq.gz,/test/data/test4_R1.fastq.gz \
    --fastq2 /test/data/test3_R2.fastq.gz,/test/data/test4_R2.fastq.gz \
    --beadstrans /scRNA/test/output/singlecell.csv \
    --ref Human \
    --name test_bcr \
    --threads 10 \
    --chain IG
```
