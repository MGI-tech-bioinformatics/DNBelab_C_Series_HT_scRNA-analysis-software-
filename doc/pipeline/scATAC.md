# 单细胞ATAC分析

## 运行 dnbc4tools atac



**工作流程如下图所示:**

 ![image-20240927163629666](https://s2.loli.net/2024/09/27/exd1OyX3n4K8LGq.png)





> [!Tip]
>
> - `$dnbc4tools`代表可执行程序的路径，通常在使用前需要将其替换为实际的安装路径。例如，如果程序安装在 /opt/software/dnbc4tools2.1.3。则对应命令
>
> ```shell
> /opt/software/dnbc4tools2.1.3/dnbc4tools atac run ...
> ```
>
> - 换行符 `\` 用于在命令行中将命令分为多行，以提高可读性。它表示命令未结束，下一行是该命令的继续。如果分析输入在一行中，则不需要使用反斜杠。

</br>
</br>

### ATAC 分析步骤

#### 第一步：准备FASTQ文件

FASTQ 文件

</br>

#### 第二步：准备参考数据库（可选）

- **基因组文件**：基因组文件应以 FASTA 格式提供，包含特定物种的完整基因组序列，包括染色体、线粒体及其他遗传信息，通常为主装配版本。这些文件为基因组分析和比对提供基础数据。
- **注释文件**：基因组注释文件应以 GTF 格式提供，包含基因组中基因、转录本、外显子及其他功能区域的详细信息。该文件标识基因的位置、类型（如“gene”、“transcript”、“exon”）及其相关属性（如“gene_id”、“gene_name”、“transcript_id”、“transcript_name”）。这些信息对理解基因组的功能和结构至关重要。

对于可从 [Ensembl 数据库](https://www.ensembl.org/index.html) 获取的物种，建议使用该处提供的文件。Ensembl 的 GTF 文件包含可选标签，便于过滤（通过 `dnbc4tools tools mkgtf`）。如果 Ensembl 无法提供所需物种的文件，则可以使用其他来源的 GTF 和 FASTA 文件。请注意，GTF 文件为必需，不支持 GFF 文件。基因组文件与注释文件需对应，GTF 文件格式要求为：对于单细胞 ATAC分析，GTF 文件至少需包含“gene”或“transcript”类型的注释。

##### 2.1 使用dnbc4tools tools mkgtf过滤GTF文件（可选）

有关GTF文件过滤的详细信息，请[参考](./scRNA.md)。

##### 2.2 **使用dnbc4tools atac mkref构建参考数据库**

在运行dnbc4tools atac run分析之前，我们需要优先构建参考数据库

需要注释文件 GTF 和参考基因组 FASTA 来构建索引文件，用于测序 reads 的比对和统计分析。以下是一个示例步骤或脚本模板：

```shell
$dnbc4tools atac mkref --fasta genome.fa --ingtf genes.gtf --species Homo_sapiens --threads 10
```

运行时打印信息

```shell
Building index for dnbc4tools atac
chromap verison: 0.2.6-r490
runMode: genomeGenerate
genomeDir: /opt/database/Mus_musculus
Mitochondrial chromosome: chrM
The remaining chromosomes are:
['chr9', 'chr12', 'chr11', 'chr15', 'chr14', 'chr16', 'chrM', 'chrX', 'chr13', 'chr10', 'chr4', 'chr8', 'chr1', 'chr19', 'chr18', 'chr3', 'chr6', 'chr7', 'chr2', 'chr5', 'chrY', 'chr17']
Analysis Complete
```

运行完成后输出：

```shell
/opt/database/Mus_musculus
├── chrom.sizes
├── gencode.vM23.primary_assembly.annotation.gtf
├── genes.filter.gtf
├── genome.fa.fai
├── genome.index
├── GRCm38.primary_assembly.genome.fa
├── promoter.bed
├── ref.json
├── tss.bed
```

其中ref.json文件中记录数据库的主要信息。

```shell
{
    "species": "Mus_musculus",
    "genome": "/opt/database/Mus_musculus/GRCm38.primary_assembly.genome.fa",
    "index": "/opt/database/Mus_musculus/genome.index",
    "chrmt": "chrM",
    "chloroplast": "None",
    "chromeSize": "/opt/database/Mus_musculus/chrom.sizes",
    "tss": "/opt/database/Mus_musculus/tss.bed",
    "promoter": "/opt/database/Mus_musculus/promoter.bed",
    "blacklist": "/opt/database/Mus_musculus/mm10.full.blacklist.bed",
    "genomesize": "mm"
}
```

</br>

#### 第三步：多样本操作（可选）

为了简化每个样本单独生成主分析流程，可以使用配置文件来生成一个包含多个样本的主流程 shell 脚本。以下是一个示例步骤或脚本模板：

```shell
$dnbc4tools atac multi --list sample.tsv --genomeDir /opt/database/Mus_musculus --threads 10
```

其中sample.tsv文件使用制表符 (\t) 分隔符。第一列包含样本名称，第二列包含文库测序数据。多个 fastq 文件应以逗号分隔，R1 和 R2 文件应以分号分隔。

```shell
$sample1 /data/sample1_R1.fq.gz;/data/sample1_R2.fq.gz 
$sample2 /data/sample2_R1.fq.gz;/data/sample2_R2.fq.gz
$sample3 /data/sample3_1_R1.fq.gz,/data/sample3_2_R1.fq.gz;/data/sample3_1_R2.fq.gz,/data/sample3_2_R2.fq.gz
```

运行完成后输出：

```shell
sample1.sh
sample2.sh
sample3.sh
```

其中文件 sample1.sh 如下：

```shell
$cat sample1.sh
/opt/software/dnbc4tools2.1.3/dnbc4tools atac run --name sample1 --fastq1 /data/sample1_R1.fq.gz --fastq2 /data/sample1_R2.fq.gz --genomeDir /opt/database/Mus_musculus --threads 10 
```

执行第四步进行主流程分析。

</br>

#### 第四步：主分析流程

ATAC 主分析流程。使用单个样本单细胞 ATAC 文库测序数据，经过过滤和比对生成所有磁珠的 fragments 文件。合并磁珠并执行 peak 调用分析，利用 peaks 区域的片段信息进行细胞识别。随后进行细胞过滤、降维和聚类，最终整合各步骤结果生成 HTML 网页报告并输出分析结果。

为单个样本生成表达矩阵，以下是一个示例步骤或脚本模板：

```shell
$dnbc4tools atac run \
		--name sample \
		--fastq1 /sample/data/test1_R1.fastq.gz,/sample/data/test2_R1.fastq.gz \
		--fastq2 /sample/data/test1_R2.fastq.gz,/sample/data/test2_R2.fastq.gz \
		--genomeDir /opt/database/Mus_musculus \
		--threads 10
```


在对试剂版本和暗反应自动检测后，软件开始运行分析，以下是一个示例:

```shell
Chemistry(darkreaction) determined in fastqR1: darkreaction
Chemistry(darkreaction) determined in fastqR2: darkreaction

2024-08-21 11:33:49
Conduct quality control for raw data, perform alignment.

2024-08-21 12:52:10
Calculating bead similarity and merging beads within the same droplet.

2024-08-21 13:14:50
Analyze fragments for peak calling.

2024-08-21 13:35:33
Generating the raw peaks matrix.

2024-08-21 14:59:27
Generating the filter peaks matrix.

2024-08-21 15:32:31
Conducting dimensionality reduction and clustering.

2024-08-21 15:49:23
Statistical analysis and report generation for results.

Analysis Finished
Elapsed Time: 4 hours 16 minutes 14 seconds
```

成功的运行会以Analysis Finished结束。