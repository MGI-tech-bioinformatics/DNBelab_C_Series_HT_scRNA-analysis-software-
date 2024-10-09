# 单细胞ATAC

## dnbc4tools atac run

用法

```shell
$dnbc4tools atac run
usage: dnbc4tools atac run [-h] 

--fastq1, --fastq2
    Multiple raw FASTQ files separated by commas and belong to the same sequencing library.
    Order of R1/R2 fastq files must be consistent.

--darkreaction
    Recommend automatic detection for settings. Ensure consistent sequencing lengths and dark
    cycles for multiple FASTQ data. Dark cycle modes can be "R1R2", "R1", "R2", "unset", etc.

--customize
    Comma-separated string value: [r1|r2|bc]:start:end:strand. Inclusive start and end, -1
    means end of read. Example: "bc:6:15,bc:22:31,r1:65:-1,r2:19:-1". "bc" indicates cell
    barcode information, "6:15" denotes positions 7 to 16 in the sequence.

optional arguments:
  -h, --help            show this help message and exit
  --name <SAMPLE_ID>    User-defined sample ID.
  --fastq1 <FQ1FILES>   The input R1 fastq files.
  --fastq2 <FQ2FILES>   The input R2 fastq files.
  --genomeDir <DATABASE>
                        Path to the directory where genome files are stored.
  --outdir <OUTDIR>     Output directory. [default: current directory]
  --threads <CORENUM>   Number of threads used for analysis. [default: 4]
  --darkreaction <DARKCYCLE>
                        Sequencing dark cycles. Automatic detection is recommended. [default: auto]
  --customize <STRUCTURE>
                        Customize read structure.
  --forcecells <CELLNUM>
                        Force pipeline to use this number of cells.
  --frags_cutoff <MIN_FRAGMENTS>
                        Filter cells with unique fragments number lower than this value. [default: 1000]
  --tss_cutoff <MIN_TSS_RATIO>
                        Filter cells with tss proportion lower than this value. [default: 0]
  --merge_cutoff <MIN_MERGE>
                        The lowest number of fragments when merging beads. [default: 1000]
  --process <ANALYSIS_STEPS>
                        Custom analysis steps enable the skipping of unnecessary steps. [default: data,decon,analysis,report]
  --bam                 The alignment process generates BAM format files, but this significantly prolongs the analysis time.
```

| 参数                                                     | 描述                                                         |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| **--name**                                               | **必需参数**，定义样本名称，与生成的HTML报告中显示的样本ID一致。 |
| **--fastq1 <br />--fastq2**                              | **必需参数**，`fastq1`和`fastq2`代表ATAC文库的R1和R2序列。多个FASTQ序列应以逗号分隔，并确保R1和R2序列的排序一致。FASTQ文件的测序模式必须相同，暗反应设置需保持一致。不同实验或不同样本的数据不得合并分析，仅同一文库的数据可以合并进行分析。 |
| **--genomeDir**                                          | **必需参数**，指定在单细胞ATAC文库准备过程中生成的参考数据库的目录。该数据库包含基因组文件、bed格式的转录起始位点文件、比对数据库、线粒体染色体名称及染色体等信息。 |
| **--outdir**                                             | **可选参数**，指定结果保存的目录。该目录的名称将基于`name`参数提供的样本ID，默认为当前目录。 |
| **--threads**                                            | **可选参数**，分析过程中使用的线程数量，增加线程数量可以加速分析。 |
| **--forcecells**                                         | **可选参数**，根据与peak重叠的fragments数量排序提取指定数量的细胞，以确定真实细胞。该值具有最高优先级。 |
| **--frags_cutoff<br />--tss_cutoff<br />--merge_cutoff** | **可选参数**，`frags_cutoff` 是细胞过滤阶段使用的最低fragments数量，默认为1000。`merge_cutoff` 是合并细胞条形码所需的最低fragments数量，默认为1000。在peak calling步骤中，仅考虑片段计数超过`merge_cutoff`的细胞。建议保持`merge_cutoff`和`frags_cutoff`一致或不高于此值。`tss_cutoff` 指的与转录起始位点区域重叠的fragments占比，低于此值的细胞将被过滤，默认为不过滤。建议使用默认参数进行分析，并在获得结果报告后再根据结果调整这些参数。 |
| **--darkreaction<br /> --customize**                     | **可选参数**，软件可自动识别文库Read1和Read2序列结构中的暗反应设置，暗反应指的是不识别碱基的生化反应，通常设置为固定碱基。识别逻辑为：软件首先检查前200,000个序列的长度来确定暗反应的存在。建议使用自动检测。暗反应模式包括“R1R2”、“R1”、“R2”、“unset”，其中“R1R2”表示R1和R2为暗反应设置。对于超出标准设置的特殊需求，用户可以直接使用`customize`定义相关信息，以逗号分隔的字符串值：[r1\|r2\|bc]:start:end，包含起始和结束位置，-1表示读取末尾。例如：“bc:6:15,bc:22:31,r1:65:-1,r2:19:-1”。“bc”表示细胞条形码信息，“6:15”表示序列的第7到16个位置。位置的数值从0开始，实际位置相差1。 |
| **--process**                                            | **可选参数**，设置分析步骤，包括：<br> - **data**：执行文库测序数据的质量控制和比对，生成raw fragments.tsv.gz文件，并过滤仅保留chromesize文件中指定的染色体信息。计算具有超过`merge_cutoff`的细胞条形码的磁珠间的Jaccard距离并绘制曲线。使用Otsu算法获得合并的最低值。合并细胞条形码并生成合并后的 all.merge.fragments.tsv.gz文件。执行peak calling分析以获取peak位置信息。<br> - **decon**：利用peak位置信息和fragments数据生成raw_peak_matrix。识别真实细胞（基于与peak重叠的片段曲线、fragments数量和与转录起始位点区域重叠的fragments占比进行过滤）。生成filter_peak_matrix。计算TSS富集分数并进行饱和度分析。<br/> - **analysis**：过滤细胞并执行降维聚类。<br/> - **report**：生成结果文件和HTML网页报告。<br/>用户可选择分析步骤跳过已完成的部分。在所有参数中，`--forcecells`、`--frags_cutoff`和`--tss_cutoff`在decon步骤中发生。调整这些参数可能会跳过data步骤。使用`--process decon, analysis, report`。参数`--darkreaction`、`--customize`、`--merge_cutoff`和`--bam`在data步骤中发生。如果需要调整，无法跳过该过程。默认使用整个过程：`--process data, decon, analysis, report`。 |
| **--bam**                                                | **标志参数**，在比对过程中生成BAM文件，但这会显著增加软件分析时间。如果不需要BAM文件，建议不要使用该参数。 |

</br>
</br>

## dnbc4tools atac mkref

用法

```shell
$dnbc4tools atac mkref
usage: dnbc4tools atac mkref [-h] 

--chrM
    Define mitochondrial chromosomes. "auto" recognizes "chrM,MT,chrMT,mt,Mt".

--prefix
    String or list of strings representing prefix(es) or full name(s) of chromosomes to keep.
    For example, "--prefix chr" selects chromosome sequences starting with "chr", or "--prefix
    1,2,3,4,5,Mt,Pt" selects chromosome sequences of Arabidopsis thaliana

--noindex
    Skip indexing step if database has been built using chromap. Only generate ref.json file.

Example:
    dnbc4tools atac mkref --fasta /database/genome.fasta --ingtf /database/genes.gtf --species
    Homo_sapiens --chrM MT --genomeDir /database

optional arguments:
  -h, --help            show this help message and exit
  --fasta <FASTA>       Path to the genome file in FASTA format.
  --ingtf <GTF>         Path to the genome annotation file in GTF format.
  --genomeDir <DATABASE>
                        Path to the directory where genome files are stored, [default: current dir].
  --species <SPECIES>   Species name, [default: undefined].
  --tag <TYPE>          Select the type to generate bed, [default: transcript].
  --chrM <Mito>         Mitochondrial chromosome name, [default: auto].
  --chloroplast <CHLOROPLAST>
                        Chloroplast chromosome names, particularly recommended for plant, such as the name "Pt".
  --prefix <CHROMOSOMES>
                        Filter chromosomes by prefix or full name.
  --noindex             Only generate ref.json without constructing genome index.
```

| 参数                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| **--fasta<br />--ingtf** | **必需参数**，提供您物种的参考基因组FASTA文件和GTF注释文件。如果Ensembl数据库中提供相应数据，推荐使用该数据库的文件。如果您的目标物种不在Ensembl中，也可使用其他来源的GTF和FASTA文件。请注意，GTF文件是必需的，且不支持GFF文件。建议使用的基因组FASTA文件应为`primary`组装版本。GTF文件的格式要求：基因组文件与注释文件需对应，GTF 文件格式要求为：对于单细胞 ATAC分析，GTF 文件至少需包含“gene”或“transcript”类型的注释。 |
| **--genomeDir**          | **可选参数**，指定存储数据库文件的目录路径，默认为当前路径。 |
| **--species**            | **可选参数**，指定用于构建参考数据库的物种名称。             |
| **--tag**                | **可选参数**，生成bed格式的转录起始位点文件时使用基因信息或转录本信息，默认使用转录本信息。 |
| **--chrM**               | **可选参数**，识别线粒体染色体名称。“auto”选项会在“chrM、MT、chrMT、mt、Mt”名称中寻找线粒体染色体名称。 |
| **--chloroplast**        | **可选参数**，设置叶绿体染色体名称，建议为植物样本设置染色体名称。若不设置线粒体和叶绿体，当线粒体和叶绿体的片段数量极高时，可能会导致在合并磁珠步骤中内存消耗过大并产生错误，以及提高了与转录起始位点区域重叠的fragments占比。 |
| **--prefix**             | **可选参数**，表示要保留的染色体前缀或全名的字符串或字符串列表。例如，`--prefix chr` 选择以“chr”开头的染色体序列，或 `--prefix 1,2,3,4,5,Mt,Pt` 选择指定的染色体。 |
| **--noindex**            | **标志参数** ，如果数据库已经使用Chromap构建，则跳过索引步骤。 |

> [!TIP]
>
> 使用Chromap进行文库构建和比对目前无法处理极大的基因组，因此某些物种无法利用该软件进行scATAC分析。未来的更新将探索超大基因组的替代比对方法。数据库库构建完成后，数据库目录中会生成一个ref.json文件以记录关键信息。
>
> ```json
> {
> "species": "Homo_sapiens",
> "genome": "$PATH/genome.fa",
> "index": "$PATH/genome.index",
> "chromeSize": "$PATH/chrom.sizes",
> "tss": "$PATH/tss.bed",
> "promoter": "$PATH/promoter.bed",
> "blacklist": "None",
> "chrmt": "chrM",
> "chloroplast": "None",
> "genomesize": "hs"
> }
> ```
>
> chromeSize文件中列出的染色体名称将包含在fragments.tsv.gz文件中进行分析。未在此文件中列出的染色体将被排除在分析之外。
>
> 自2.1.2版本，已移除blacklist参数不再强制要求存在黑名单文件。如有需要，可以手动添加。此参数不会影响分析结果。黑名单区域的片段数量将记录在metadata文件output/singlecell.csv中的blacklist_region_fragments列中。
>
> genomesize值用于MACS2 peak calling 分析。MACS2对某些物种有特殊标识符，例如“hs”表示Homo sapiens。

</br>
</br>

## dnbc4tools atac multi

用法

```shell
$dnbc4tools atac multi
usage: dnbc4tools atac multi [-h] 

All samples should be from the same species or the same reference database.
--list
    Generate a two-column list with tab (\t) separators. R1 and R2 reads should be separated
    by semicolons, and multiple FASTQ files should be separated with commas.

optional arguments:
  -h, --help            show this help message and exit
  --list <LIST>         sample list.
  --outdir <OUTDIR>     Output diretory, [default: current directory].
  --threads <CORENUM>   Number of threads used for analysis, [default: 4].
  --genomeDir <DATABASE>
                        Path to the directory where genome files are stored.
```

| 参数   | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| --list | **必需参数**，文件使用制表符 (\t) 分隔符。第一列包含样本名称，第二列包含 ATAC 文库测序数据。多个 fastq 文件应以逗号分隔，R1 和 R2 文件应以分号分隔。 |

其他参数参考dnbc4tools atac run.