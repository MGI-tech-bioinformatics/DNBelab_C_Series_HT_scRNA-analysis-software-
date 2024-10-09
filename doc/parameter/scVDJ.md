# 单细胞VDJ

## dnbc4tools vdj run

用法

```shell
$dnbc4tools vdj run
usage: dnbc4tools vdj run [-h] 

--fastq1, --fastq2
    Multiple raw FASTQ files separated by commas and belong to the same sequencing library.
    Order of R1/R2 fastq files must be consistent.

--ref
    Provide reference databases for human and mouse for analysis. Other species are not
    supported

--beadstrans
    Required parameters. The analysis of the 5' scRNA must be completed first. In the analysis
    results directory, there should be a file named singlecell.csv that records the
    correspondence between cells and beads.

--darkreaction
    Recommend automatic detection for settings. Ensure consistent sequencing lengths and dark
    cycles for multiple FASTQ data. Dark cycle modes can be "R1" and "unset"

optional arguments:
  -h, --help            show this help message and exit
  --name <SAMPLE_ID>    User-defined sample ID.
  --fastq1 <FQ1FILES>   The input R1 fastq files.
  --fastq2 <FQ2FILES>   The input R2 fastq files.
  --outdir <OUTDIR>     Output directory, [default: current directory].
  --ref <REF>           Set the reference database, choose from 'human' and 'mouse'.
  --chain <CHAIN>       Chain type to display metrics for: 'TR' for T cell receptors, 'IG' for B cell receptors.
  --beadstrans <SINGLECELL>
                        Beads converted into cells file in rna summary file 'output/singlecell.csv'.
  --threads <CORENUM>   Number of threads used for analysis, [default: 10].
  --darkreaction <DARKCYCLE>
                        Sequencing dark cycles. Automatic detection is recommended, [default: auto].
  --customize <STRUCTURE>
                        Customize whitelist and readstructure files in JSON format.
  --process <ANALYSIS_STEPS>
                        Custom analysis steps enable the skipping of unnecessary steps, [default: data,assembly,filter,report].
  --nornafilter         Retain cells including those not identified in the corresponding 5' Gene Expression dataset.
  --singleEnd           Assemble using single-end data.
```

| 参数                                | 描述                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| **--name**                          | **必需参数**，定义样本名称，与生成的 HTML 报告中的样本 ID 一致。 |
| **--fastq1 <br />--fastq2**         | **必需参数**，`fastq1` 和 `fastq2` 代表 VDJ 文库的 R1 和 R2 序列。多个 FASTQ 文件应以逗号分隔，确保 R1 和 R2 序列的排序一致。测序模式必须相同，暗反应设置需保持一致。不同实验或样本的数据不得合并，仅同一文库的数据可合并分析。 |
| **--ref**                           | **必需参数**，软件自带人类和小鼠的参考数据库，可直接使用与物种对应的数据库。 |
| **--chain**                         | **必需参数**，分析特定链的类型。可接受的值为：“TR”表示 T 细胞受体；“IG”表示 B 细胞受体。 |
| **--beadstrans**                    | **必需参数**，需先完成 5' scRNA 的分析，结果目录中应有一个名为“singlecell.csv”的文件，记录细胞与磁珠的对应关系。 |
| **--outdir**                        | **可选参数**，指定结果保存的目录，默认为当前目录。目录名称基于 `name` 参数提供的样本 ID。 |
| **--threads**                       | **可选参数**，分析过程中使用的线程数量，增加线程可加速分析。 |
| **--darkreaction<br />--customize** | **可选参数**，软件可自动识别文库 Read1 的暗反应设置，暗反应指不识别碱基的生化反应，通常设置为固定碱基。识别逻辑为：检查前 200,000 个序列的长度来确定暗反应的存在。建议使用自动检测。暗反应模式包括“R1”、“unset”，其中“R1”表示 R1 为暗反应设置。用户可使用 `customize` 修改白名单 JSON 文件中的相关信息，详细信息见文档。 |
| **--process**                       | **可选参数**，设置分析步骤，包括：<br> - **data**：执行文库测序数据的质量控制，利用 5' 转录组结果合并磁珠，比对 VDJ 基因区域并提取对应的 reads。<br> - **assembly**：利用 VDJ 基因区域的 reads 按照每个细胞分别进行重头组装，将组装得到的 contigs 使用IMGT数据库进行注释。<br/> - **filter**：根据注释结果和 5' 转录组的细胞获取情况对细胞进行过滤并生成 clonotype 信息。<br/> - **report**：生成结果文件和HTML网页报告。<br />用户可选择分析步骤跳过已完成的部分。 |
| **--nornafilter**                   | **标志参数**，不使用 5' 转录组的细胞获取情况对细胞进行过滤。 |
| **--singleEnd**                     | **标志参数**，当测序时 Read1 仅测序了cell barcode和umi信息，而未测序插入片段时，仅使用 Read2 数据进行组装。 |

