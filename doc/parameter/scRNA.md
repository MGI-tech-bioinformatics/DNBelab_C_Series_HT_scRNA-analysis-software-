# 单细胞RNA

## dnbc4tools rna run

用法

```shell
$dnbc4tools rna run -h
usage: dnbc4tools rna run [-h]

--cDNAfastq1, --cDNAfastq2, --oligofastq1, --oligofastq2
    Multiple raw FASTQ sequences, separated by commas, should have consistent ordering of the
    cDNA or oligo R1/R2 FASTQ files.

--chemistry, --darkreaction
    Recommend automatic detection for settings. When multiple FASTQ files are used, ensure
    that the cDNA or oligo libraries have consistent dark cycles. If manual configuration of
    the reagent version and dark cycles is necessary, both parameters should be set together.
    Reagent versions available are "scRNAv1HT", "scRNAv2HT", "scRNAv3HT" and "scRNA5Pv1". Dark
    cycles should be separated by comma, e.g., "R1,R1R2", "R1,R1", "unset,unset", etc.

optional arguments:
  -h, --help            show this help message and exit
  --name <SAMPLE_ID>    User-defined sample ID.
  --cDNAfastq1 <FQ1FILES>
                        Paths to the raw R1 FASTQ files of the cDNA library.
  --cDNAfastq2 <FQ2FILES>
                        Paths to the raw R2 FASTQ files of the cDNA library.
  --oligofastq1 <FQ1FILES>
                        Paths to the raw R1 FASTQ files of the oligo library.
  --oligofastq2 <FQ2FILES>
                        Paths to the raw R2 FASTQ files of the oligo library.
  --genomeDir <DATABASE>
                        Path to the directory containing genome files.
  --outdir <OUTDIR>     Output directory, [default: current directory].
  --threads <CORENUM>   Number of threads used for analysis, [default: 4].
  --calling_method <CELLCALLING>
                        Cell calling method, choose from barcoderanks and emptydrops, [default: emptydrops].
  --expectcells <CELLNUM>
                        Expected number of recovered cells, [default: 3000].
  --forcecells <CELLNUM>
                        Force pipeline to use a specific number of cells.
  --chemistry <CHEMISTRY>
                        Chemistry version. Automatic detection recommended , [default: auto].
  --darkreaction <DARKCYCLE>
                        Sequencing dark cycles. Automatic detection recommended, [default: auto].
  --customize <STRUCTURE>
                        Customize whitelist and readstructure files in JSON format for cDNA and oligo.
  --process <ANALYSIS_STEPS>
                        Custom analysis steps to skip unnecessary processes, [default: data,count,analysis,report].
  --no_introns          Intron reads are not included in the expression matrix.
  --end5                Perform 5'-end single-cell transcriptome analysis.
```

| 参数                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **--name**                                                   | **必需参数**，定义样本名称，与生成的HTML报告中显示的样本ID一致。 |
| **--cDNAfastq1 <br />--cDNAfastq2<br />--oligofastq1<br />--oligofastq2** | **必需参数**，`cDNAfastq1`和`cDNAfastq2`代表cDNA文库的R1和R2序列，`oligofastq1`和`oligofastq2`则代表oligo文库的R1和R2序列。多个FASTQ序列应以逗号分隔，并确保R1和R2序列的排序一致。FASTQ文件的测序模式必须相同，暗反应设置需保持一致。不同实验或不同样本的数据不得合并分析，仅同一文库的数据可以合并进行分析。 |
| **--genomeDir**                                              | **必需参数**，指定在单细胞RNA文库准备过程中生成的参考数据库的目录。该目录应包含基因组文件、GTF格式的注释文件、STAR比对数据库（版本2.7.2b）、线粒体染色体的名称列表及线粒体基因列表。 |
| **--outdir**                                                 | **可选参数**，指定结果保存的目录。该目录的名称将基于`name`参数提供的样本ID，默认为当前目录。 |
| **--threads**                                                | **可选参数**，分析过程中使用的线程数量，增加线程数量可以加速分析。 |
| **--calling_method<br />--expectcells<br /> --forcecells**   | **可选参数**，`calling_method`用于在分析过程中识别真实细胞。可选方法包括“emptydrops”和“barcoderanks”，默认使用“emptydrops”。在“emptydrops”算法中，首先根据预期细胞数量参数（`expectcells`，建议填写为投入有效细胞数量的50%；若未提供投入细胞数，建议使用默认值）进行初步筛选，捕获具有高UMI（唯一分子标识符）区域。UMI低于设定阈值（默认1000，可通过`minumi`参数调整）的细胞将被过滤。最后，介于两者之间与背景表达矩阵显著不同的细胞将被认定为真实细胞。“barcoderanks”方法则根据UMI排序曲线确定真实细胞，将曲线的拐点作为阈值，高于该点的细胞即被视为真实细胞。`forcecells`选项允许用户根据UMI的排序结果，选择并提取排序靠前的特定数量的细胞。 |
| **--chemistry<br /> --darkreaction<br /> --customize**       | **可选参数**，软件可根据cDNA和oligo文库的Read1、Reads2序列结构自动识别试剂类型和暗反应设置。暗反应指的是不识别碱基的生化反应，通常设置为固定碱基。软件逻辑如下：首先检查前200,000个序列的长度以确定是否存在暗反应。如果未检测到暗反应，则使用固定碱基序列信息及不同试剂版本对应的位置来提供适合的白名单结构文件。如果自动识别失败或不准确，可以使用`chemistry`和`darkreaction`进行手动设置。`chemistry`选项包括“scRNAv1HT”、“scRNAv2HT”、“scRNAv3HT”和“scRNA5Pv1”，而`darkreaction`选项则用逗号分隔cDNA和oligo文库的设置，例如“R1,R1R2”、“R1,R1”、“unset,unset”等，其中“R1,R1R2”表示为cDNA文库的R1和oligo文库的R1R2设置的暗反应。对于超出标准设置的特殊需求，用户可以直接使用`customize`修改白名单JSON文件中的相关信息。有关JSON文件格式的详细信息，请参见相关文档。 |
| **--process**                                                | **可选参数**，设置分析步骤，包括：<br> - **data**：对cDNA文库数据执行质量控制（QC）和比对，根据注释信息确定比对位置信息，并生成`final_sorted.bam`文件。同时，对oligo文库数据执行质量控制（QC），并根据R1R2端信息生成`CB_UB_count.txt`文件。<br> - **count**：基于cDNA和oligo信息计算细胞条形码之间的相似性，将高相似性的细胞条形码合并为同一细胞，生成合并的原始表达矩阵，利用细胞识别方法识别真实细胞，生成过滤后的表达矩阵，并进行饱和度计算。<br> - **analysis**：过滤细胞，并对过滤后的表达矩阵进行降维、聚类和注释。<br> - **report**：生成结果文件和HTML网页报告。<br />用户可选择分析步骤跳过已完成的部分。在所有参数中，`calling_method`、`expectcells`和`forcecells`参数在count步骤中使用。调整这些参数可能会跳过data步骤，建议使用`--process count,analysis,report`。参数`chemistry`、`darkreaction`、`customize`、`no_introns`和`end5`在data步骤中使用。如需调整，则无法跳过流程，需使用完整流程：`--process data,count,analysis,report`。 |
| **--no_introns**                                             | **标志参数**，在分析过程中过滤掉来自内含子区域的reads，仅保留来自外显子区域的reads以进行表达量化。 |
| **--end5**                                                   | **标志参数**，运行5'端转录组数据分析。                       |

</br>
</br> 

## dnbc4tools rna mkref

用法

```shell
$dnbc4tools rna mkref -h
usage: dnbc4tools rna mkref [-h] 
--species
    Specify the species name for constructing the reference database. For cell annotation
    analysis, only "Homo_sapiens", "Human", "Mus_musculus", and "Mouse" are valid options.

--chrM
    Identify mitochondrial chromosomes. "Auto" recognizes "chrM, MT, chrMT, mt, Mt". Genes
    located on mitochondria will be automatically identified, generating the file
    "mtgene.list".

--noindex
    Skip indexing step if database has already been constructed using scSTAR.

Example:
    dnbc4tools rna mkref --fasta /database/genome.fasta --ingtf /database/genes.gtf --species
    Homo_sapiens --chrM MT --genomeDir /database --threads 10

optional arguments:
  -h, --help            show this help message and exit
  --fasta <FASTA>       Path to the genome file in FASTA format.
  --ingtf <GTF>         Path to the genome annotation file in GTF format.
  --species <SPECIES>   Species name, [default: undefined].
  --chrM <MT>           Mitochondrial chromosome name, [default: auto].
  --genomeDir <DATABASE>
                        Path to the directory where database files will be stored, [default: current dir].
  --limitram <MEMORY>   Maximum available RAM (bytes) for genome index generation.
  --threads <CORENUM>   Number of threads used for analysis, [default: 4].
  --noindex             Only generate ref.json without constructing the genome index.
```

| 参数                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| **--fasta<br />--ingtf** | **必需参数**，提供您物种的参考基因组FASTA文件和GTF注释文件。如果Ensembl数据库中提供相应数据，推荐使用该数据库的文件。如果您的目标物种不在Ensembl中，也可使用其他来源的GTF和FASTA文件。请注意，GTF文件是必需的，且不支持GFF文件。建议使用的基因组FASTA文件应为`primary`组装版本。GTF文件的格式要求：对于单细胞RNA分析，GTF文件必须至少包含“gene”或“transcript”类型以及“exon”类型的注释。属性中至少应包含“gene_id”或“gene_name”以及“transcript_id”或“transcript_name”。 |
| **--species**            | **可选参数**，指定用于构建参考数据库的物种名称。在细胞注释分析中，仅“Homo_sapiens”、“Human”、“Mus_musculus”和“Mouse”是有效选项。 |
| **--chrM**               | **可选参数**，识别线粒体染色体名称。“auto”选项会在“chrM、MT、chrMT、mt、Mt”名称中寻找线粒体染色体名称。如果存在线粒体染色体名称，位于线粒体上的基因将被自动获取，并生成“mtgene.list”文件，否则为“None”。 |
| **--genomeDir**          | **可选参数**，指定存储数据库文件的目录路径，默认为当前路径。 |
| **--limitram**           | **可选参数**，用于基因组索引生成的最大可用RAM（字节数）。    |
| **--threads**            | **可选参数**，分析过程中使用的线程数量，增加线程数量可以加速分析。 |
| **--noindex**            | **标志参数** ，如果数据库已经使用STAR构建，则跳过索引步骤。  |

> [!TIP]
>
> 对于具有众多且不同染色体大小的基因组，数据库构建已调整为自动确定`genomeSAindexNbases`和`genomeChrBinNbits`的值。
>
> 数据库构建完成后，将在数据库目录中生成`ref.json`文件以记录关键信息。
>
> ```json
> {
> "species": "Homo_sapiens",
> "genome": "$PATH/genome.fa",
> "gtf": "$PATH/genes.filter.gtf",
> "genomeDir": "$PATH",
> "chrmt": "chrM",
> "mtgenes": "$PATH/mtgene.list"
> }
> ```

</br>
</br> 

## dnbc4tools rna multi

用法

```shell
$dnbc4tools rna multi -h
usage: dnbc4tools rna multi [-h] 

All samples should be from the same species or the same reference database.
--list
    Generate a three-column list with tab (\t) separators. First column contains sample names,
    second contains cDNA library sequencing data, and third contains oligo library sequencing
    data. Multiple fastq files should be separated by commas, and R1 and R2 files should be
    separated by semicolons.

optional arguments:
  -h, --help            show this help message and exit
  --list <LIST>         sample list.
  --genomeDir <DATABASE>
                        Path to the directory containing genome files.
  --outdir <OUTDIR>     Output directory, [default: current directory].
  --threads <CORENUM>   Number of threads used for analysis, [default: 4].
  --calling_method <CELLCALLING>
                        Cell calling method, choose from barcoderanks and emptydrops, [default: emptydrops].
  --expectcells <CELLNUM>
                        Expected number of recovered beads, [default: 3000].
  --no_introns          Intron reads are not included in the expression matrix.
  --end5                Perform 5'-end single-cell transcriptome analysis.
```

| 参数   | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| --list | **必需参数**，文件使用制表符 (\t) 分隔符。第一列包含样本名称，第二列包含 cDNA 文库测序数据，第三列包含 oligo 文库测序数据。多个 fastq 文件应以逗号分隔，R1 和 R2 文件应以分号分隔。 |

其他参数参考dnbc4tools rna run.