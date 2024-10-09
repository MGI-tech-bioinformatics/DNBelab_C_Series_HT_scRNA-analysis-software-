# 单细胞RNA分析

## 运行 dnbc4tools rna



**工作流程如下图所示:**

 ![image-20240926152210545](https://s2.loli.net/2024/09/26/uKTXv7Q2miNbz1S.png)





> [!Tip]
>
> - `$dnbc4tools`代表可执行程序的路径，通常在使用前需要将其替换为实际的安装路径。例如，如果程序安装在 /opt/software/dnbc4tools2.1.3。则对应命令
>
> ```shell
> /opt/software/dnbc4tools2.1.3/dnbc4tools rna run ...
> ```
>
> - 换行符 `\` 用于在命令行中将命令分为多行，以提高可读性。它表示命令未结束，下一行是该命令的继续。如果分析输入在一行中，则不需要使用反斜杠。

</br>
</br>

### RNA 分析步骤

#### 第一步：准备FASTQ文件

FASTQ 文件

</br>

#### 第二步：准备参考数据库（可选）

- **基因组文件**：基因组文件应以 FASTA 格式提供，包含特定物种的完整基因组序列，包括染色体、线粒体及其他遗传信息，通常为主装配版本。这些文件为基因组分析和比对提供基础数据。
- **注释文件**：基因组注释文件应以 GTF 格式提供，包含基因组中基因、转录本、外显子及其他功能区域的详细信息。该文件标识基因的位置、类型（如“gene”、“transcript”、“exon”）及其相关属性（如“gene_id”、“gene_name”、“transcript_id”、“transcript_name”）。这些信息对理解基因组的功能和结构至关重要。

对于可从 [Ensembl 数据库](https://www.ensembl.org/index.html) 获取的物种，建议使用该处提供的文件。Ensembl 的 GTF 文件包含可选标签，便于过滤（通过 `dnbc4tools tools mkgtf`）。如果 Ensembl 无法提供所需物种的文件，则可以使用其他来源的 GTF 和 FASTA 文件。请注意，GTF 文件为必需，不支持 GFF 文件。基因组文件与注释文件需对应，GTF 文件格式要求为：对于单细胞 RNA 分析，GTF 文件至少需包含“gene”或“transcript”类型以及“exon”类型的注释，并且属性中必须包含“gene_id”或“gene_name”以及“transcript_id”或“transcript_name”。

##### 2.1 使用dnbc4tools tools mkgtf过滤GTF文件（可选）

从 ENSEMBL 和 UCSC 等网站下载的 GTF 文件通常包含多种基因类型的基因。选择您研究中比较感兴趣的基因类型，过滤部分基因类型可以减少重叠的基因注释。与多个基因非唯一比对的 reads 会被过滤。

我们提供了基因类型数量统计、基因类型过滤和校正 GTF 文件的功能。

- **基因类型数量统计**（可选）

  以下是一个示例步骤或脚本模板：
  
  ```shell
  $dnbc4tools tools mkgtf --action stat --ingtf genes.gtf --output gtfstat.txt --type gene_biotype
  ```

  在上面的命令中，需要查看 GTF 文件中的 tag 确定 `type` 的类型。

  ![image-20240927111652480](https://s2.loli.net/2024/10/09/afGqtQocTE9h3uR.png)
  
  
  
  输出示例：成功运行后，输出文件，以下是一个示例：
  
  ```shell
  $cat gtf_type.txt
  Type    Count
  protein_coding  20006
  lncRNA  17755
  processed_pseudogene    10159
  unprocessed_pseudogene  2605
  misc_RNA        2221
  snRNA   1910
  miRNA   1879
  TEC     1056
  transcribed_unprocessed_pseudogene      950
  snoRNA  943
  transcribed_processed_pseudogene        503
  rRNA_pseudogene 497
  IG_V_pseudogene 187
  transcribed_unitary_pseudogene  146
  IG_V_gene       145
  TR_V_gene       106
  unitary_pseudogene      97
  TR_J_gene       79
  rRNA    53
  polymorphic_pseudogene  50
  scaRNA  49
  IG_D_gene       37
  TR_V_pseudogene 33
  Mt_tRNA 22
  pseudogene      19
  IG_J_gene       18
  IG_C_gene       14
  IG_C_pseudogene 9
  ribozyme        8
  TR_C_gene       6
  sRNA    5
  TR_D_gene       4
  TR_J_pseudogene 4
  IG_J_pseudogene 3
  translated_processed_pseudogene 2
  translated_unprocessed_pseudogene       2
  Mt_rRNA 2
  scRNA   1
  vault_RNA       1
  IG_pseudogene   1
  ```



- **校正 GTF 文件**（可选）

  GTF 文件格式要求为：对于单细胞 RNA 分析，GTF 文件至少需包含“gene”或“transcript”类型以及“exon”类型的注释，并且属性中必须包含“gene_id”或“gene_name”以及“transcript_id”或“transcript_name”。存在内容缺失的 GTF 文件会导致主分析流程无法注释而报错。

  以下是一个示例步骤或脚本模板：
  
  ```shell
  $dnbc4tools tools mkgtf --action check --ingtf genes.gtf --output corrected.gtf
  ```

  运行时打印信息
  
  ```shell
  Start checking...
  
  ==================================================
            Summary of Missing Information
  ==================================================
  Missing gene lines:            0
  Total gene lines:              38406
  Missing transcript lines:      0
  Total transcript lines:        217193
  Total exon lines:              1493173
  ==================================================
  Warning:   <chr4:88520998-88523776> matches more than multiple geneID <"PYURF", "PIGY">, please check.
  Warning:   <chr16:69118010-69132588> matches more than multiple geneID <"CHTF8", "DERPC">, please check.
  Warning:   <chrX:22259797-23293146> matches more than multiple geneID <"ENSG00000289638", "ENSG00000289084">, please check.
  
  Writting new gtf to "/opt/database/Homo_sapiens/corrected.gtf"
  Complete
  ```

  软件会主动填补 gene 行和 transcript 行缺失的信息，会根据 gene_id 和 gene_name 以及 transcript_id 和 transcript_name 互相填补。
  
  警告信息提示该位置存在多个基因信息，在分析时会导致过滤。



- **基因类型过滤**

  以下是一个示例步骤或脚本模板：
  
  ```shell
  $dnbc4tools tools mkgtf --ingtf genes.gtf --output genes.filter.gtf --type gene_biotype
  ```

  命令中我们可以添加参数 `include` 来获取需要的 gene 类型，默认是下面列出的基因型：
  
  - protein_coding
  - lncRNA/lincRNA
  - antisense
  - IG_V_gene
  - IG_LV_gene
  - IG_D_gene
  - IG_J_gene
  - IG_C_gene
  - IG_V_pseudogene
  - IG_J_pseudogene
  - IG_C_pseudogene
  - TR_V_gene
  - TR_D_gene
  - TR_J_gene
  - TR_C_gene

  例如应用以上筛选来过滤，可以使用默认选项，也可以使用如下命令：
  
  ```shell
  $dnbc4tools tools mkgtf --ingtf genes.gtf --output genes.filter.gtf --type gene_biotype \
  			--include protein_coding,lncRNA,lincRNA,antisense,IG_V_gene,IG_LV_gene \
  			IG_J_gene,IG_C_gene,IG_V_pseudogene,IG_J_pseudogene,IG_C_pseudogene \
  			TR_V_gene,TR_D_gene,TR_J_gene,TR_C_gene
  ```
  
  这将从原始未过滤的 GTF 文件生成一个过滤的 GTF 文件。在输出文件中，其他基因类型被排除在 GTF 注释之外。

</br>

##### 2.2 **使用dnbc4tools rna mkref构建参考数据库**

在运行dnbc4tools rna run分析之前，我们需要优先构建参考数据库

需要注释文件 GTF 和参考基因组 FASTA 来构建索引文件，用于测序 reads 的比对和注释。以下是一个示例步骤或脚本模板：

```shell
$dnbc4tools rna mkref --fasta genome.fa --ingtf genes.gtf --species Homo_sapiens --threads 10
```

运行时打印信息，以下是一个示例：

```shell
STAR verison: 2.7.2b
runMode: genomeGenerate
runThreadN: 10
limitGenomeGenerateRAM: 125000000000
genomeSAindexNbases: 14
genomeChrBinNbits: 18
genomeDir: /opt/database/Homo_sapiens
fasta: /opt/database/Homo_sapiens/GRCh38.primary_assembly.genome.fa
gtf: /opt/database/Homo_sapiens/genes.filter.gtf
2024-3-28  9:28:34 ..... started STAR run
Mar 28 09:28:34 ... starting to generate Genome files
Mar 28 09:29:51 ... starting to sort Suffix Array. This may take a long time...
Mar 28 09:30:07 ... sorting Suffix Array chunks and saving them to disk...
Mar 28 10:29:32 ... loading chunks from disk, packing SA...
Mar 28 10:30:58 ... finished generating suffix array
Mar 28 10:30:58 ... generating Suffix Array index
Mar 28 10:35:10 ... completed Suffix Array index
Mar 28 10:35:10 ..... processing annotations GTF
Mar 28 10:35:41 ..... inserting junctions into the genome indices
Mar 28 10:39:43 ... writing Genome to disk ...
Mar 28 10:40:13 ... writing Suffix Array to disk ...
Mar 28 10:44:01 ... writing SAindex to disk
Mar 28 10:44:19 ..... finished successfully
Analysis Complete
```

运行完成后输出：

```shell
/opt/database/Homo_sapiens
├── chrLength.txt
├── chrNameLength.txt
├── chrName.txt
├── chrStart.txt
├── exonGeTrInfo.tab
├── exonInfo.tab
├── gencode.v32.primary_assembly.annotation.gtf
├── geneInfo.tab
├── genes.filter.gtf
├── Genome
├── GRCh38.primary_assembly.genome.fa
├── genomeParameters.txt
├── Log.out
├── mtgene.list
├── ref.json
├── SA
├── SAindex
├── sjdbInfo.txt
├── sjdbList.fromGTF.out.tab
├── sjdbList.out.tab
└── transcriptInfo.tab
```

其中ref.json文件中记录数据库的主要信息。

```shell
{
 "species": "Homo_sapiens",
 "genome": "/opt/database/Homo_sapiens/GRCh38.primary_assembly.genome.fa",
 "gtf": "/opt/database/Homo_sapiens/genes.filter.gtf",
 "genomeDir": "/opt/database/Homo_sapiens",
 "chrmt": "chrM",
 "mtgenes": "/opt/database/Homo_sapiens/mtgene.list"
}
```

</br>

#### 第三步：多样本操作（可选）

为了简化每个样本单独生成主分析流程，可以使用配置文件来生成一个包含多个样本的主流程 shell 脚本。以下是一个示例步骤或脚本模板：

```shell
$dnbc4tools rna multi --list sample.tsv --genomeDir /opt/database/Homo_sapiens --threads 10
```

其中sample.tsv文件使用制表符 (\t) 分隔符。第一列包含样本名称，第二列包含 cDNA 文库测序数据，第三列包含寡核苷酸文库测序数据。多个 fastq 文件应以逗号分隔，R1 和 R2 文件应以分号分隔。

```shell
$sample1 /data/cDNA1_R1.fq.gz;/data/cDNA1_R2.fq.gz /data/oligo1_R1.fq.gz,/data/oligo4_R1.fq.gz;/data/oligo1_R2.fq.gz,/data/oligo4_R2.fq.gz 
$sample2 /data/cDNA2_R1.fq.gz;/data/cDNA2_R2.fq.gz /data/oligo2_R1.fq.gz;/data/oligo2_R2.fq.gz 
$sample3 /data/cDNA3_R1.fq.gz;/data/cDNA3_R2.fq.gz /data/oligo3_R1.fq.gz;/data/oligo3_R2.fq.gz
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
/opt/software/dnbc4tools2.1.3/dnbc4tools rna run --name sample1 --cDNAfastq1 /data/cDNA1_R1.fq.gz --cDNAfastq2 /data/cDNA1_R2.fq.gz --oligofastq1 /data/oligo1_R1.fq.gz,/data/oligo4_R1.fq.gz --oligofastq2 /data/oligo1_R2.fq.gz,/data/oligo4_R2.fq.gz --genomeDir /database/scRNA/Mus_musculus/mm10 --threads 10 
```

执行第四步进行主流程分析。

</br>

#### 第四步：主分析流程

RNA 主分析流程。处理单个样本单细胞 RNA 的 cDNA 和 oligo 文库测序数据。该流程包括质量控制、比对和功能区域注释。随后，系统将合并磁珠以识别细胞，并生成原始基因表达矩阵及过滤后的基因表达矩阵。接下来，分析将对该矩阵进行细胞过滤、降维、聚类和注释，最终生成 HTML 格式的报告并输出分析结果。

为单个样本生成表达矩阵，以下是一个示例步骤或脚本模板：

```shell
$dnbc4tools rna run \
		--name sample \
		--cDNAfastq1 /data/sample_cDNA_R1.fastq.gz \
		--cDNAfastq2 /data/sample_cDNA_R2.fastq.gz \
		--oligofastq1 /data/sample_oligo1_1.fq.gz,/data/sample_oligo2_1.fq.gz \
		--oligofastq2 /data/sample_oligo1_2.fq.gz,/data/sample_oligo2_2.fq.gz \
		--genomeDir /opt/database/Homo_sapiens \
		--threads 10
```


在对试剂版本和暗反应自动检测后，软件开始运行分析，以下是一个示例：

```shell
Chemistry(darkreaction) determined in oligoR1: darkreaction
Chemistry(darkreaction) determined in oligoR2: nodarkreaction
Chemistry(darkreaction) determined in cDNAR1 : darkreaction

2024-09-23 09:23:17
Conduct quality control for cDNA library barcoding, perform alignment and annotation of gene regions.

2024-09-23 09:23:17
Perform quality control for oligo library barcodes.

2024-09-23 10:42:23
Calculating bead similarity and merging beads within the same droplet.

2024-09-23 10:50:37
Generating the raw expression matrix.

2024-09-23 10:54:38
Generating the filtered expression matrix.

2024-09-23 10:57:48
Conducting dimensionality reduction and clustering.

2024-09-23 10:59:55
Statistical analysis and report generation for results.

Analysis Finished
Elapsed Time: 1 hours 38 minutes 51 seconds
```

成功的运行会以Analysis Finished结束。