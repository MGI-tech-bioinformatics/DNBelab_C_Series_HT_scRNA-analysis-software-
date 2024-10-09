# 单细胞VDJ分析

## 运行 dnbc4tools vdj



**工作流程如下图所示:**

 ![image-20240927180104002](https://s2.loli.net/2024/09/27/WHFIaNpLV8xu4Pi.png)





> [!Tip]
>
> - `$dnbc4tools`代表可执行程序的路径，通常在使用前需要将其替换为实际的安装路径。例如，如果程序安装在 /opt/software/dnbc4tools2.1.3。则对应命令
>
> ```shell
> /opt/software/dnbc4tools2.1.3/dnbc4tools vdj run ...
> ```
>
> - 换行符 `\` 用于在命令行中将命令分为多行，以提高可读性。它表示命令未结束，下一行是该命令的继续。如果分析输入在一行中，则不需要使用反斜杠。


</br>
</br>

### VDJ 分析步骤

#### 第一步：5端转录组分析

转录组分析请参考dnbc4tools rna run

5端转录组分析主流程分析需在单细胞RNA主流程分析基础上添加参数```--end5```。

为单个样本生成表达矩阵，以下是一个示例步骤或脚本模板：

```shell
$dnbc4tools rna run \
		--name sample_5rna \
		--cDNAfastq1 /data/sample_cDNA_R1.fastq.gz \
		--cDNAfastq2 /data/sample_cDNA_R2.fastq.gz \
		--oligofastq1 /data/sample_oligo1_1.fq.gz,/data/sample_oligo2_1.fq.gz \
		--oligofastq2 /data/sample_oligo1_2.fq.gz,/data/sample_oligo2_2.fq.gz \
		--genomeDir /opt/database/Homo_sapiens \
		--threads 10 \
		--end5
```

</br>

#### 第二步：准备文件

FASTQ 文件

对应样本5端转录组分析结果目录中 *singlecell.csv*文件，分析内容包括 CELL 列和 BARCODE 列对应的合并信息，以及 is_cell_barcode 列指示的 5' 端鉴定的细胞（1 表示是细胞，0 表示不是细胞）。

参考示例文件内容：

```shell
CELL,Raw,GENE,UMI,GnReads,is_cell_barcode,BARCODE
CELL2564_N5,382911,9257,116043,280730,1,ACTGTCGGCGCCGCAGGAGC;AGCCGCTCTATACGCCGATT;AGTCGGTGAGGCTTAGATCT;CACACTATACGTTGGATCCT;TAAGGAGAAGTCGACACGAT
CELL85702_N1,369722,9278,115657,281513,1,TTCCTCATAGAGCTATGCTT
CELL2599_N6,367948,8384,109128,278576,1,ACTTCGGATACTTCAATAAT;AGATCTCTTGAGATCGCTGT;AGTTCACTCAACCATCGGAA;CTGGCTTCCGGGAGCAACAG;GAACGTGAAGCAGGCGTACT;GAATTCTCGGATATCACGAA
CELL61325_N1,333844,8020,102564,244945,1,GCTCGTTAGTTACGTTATGG
CELL624_N2,356813,8266,101929,262750,1,AAGTAAGCGTGCGAATGACT;GTGTCACGAGGGCACTACTG
```

</br>

#### 第三步：主分析流程

VDJ 主分析流程。使用单细胞 VDJ 文库测序数据和对应样本的 5' 转录组分析结果。首先对数据进行过滤，利用 5' 转录组结果合并磁珠，接着比对 VDJ 基因区域并提取对应的 reads，进行从头组装并注释。根据组装注释结果和 5' 转录组的细胞获取情况进行细胞过滤，最后整合各步骤结果生成 HTML 网页报告并输出分析结果。



为单个样本运行TCR分析，以下是一个示例步骤或脚本模板：

```shell
$dnbc4tools vdj run \
		--name sample_tcr \
		--fastq1 /data/sample_tcr_R1.fastq.gz \
		--fastq2 /data/sample_tcr_R2.fastq.gz \
		--ref human \
		--chain TR \
		--beadstrans /sample_5rna/output/singlecell.csv \
		--threads 10
```

为单个样本运行BCR分析，以下是一个示例步骤或脚本模板：

```shell
$dnbc4tools vdj run \
		--name sample_bcr \
		--fastq1 /data/sample_tcr_R1.fastq.gz \
		--fastq2 /data/sample_tcr_R2.fastq.gz \
		--ref human \
		--chain IG \
		--beadstrans /sample_5rna/output/singlecell.csv \
		--threads 10
```


在对暗反应自动检测后，软件开始运行分析，以下是一个示例：

```shell
Chemistry(darkreaction) determined in fastqR1: darkreaction

2024-09-06 23:08:26
Barcode Preprocessing and Obtaining VDJ Region Sequences.

2024-09-07 00:15:39
Sequence Assembly and annotation of VDJ Gene Segments.

2024-09-07 05:40:40
Cell calling and generate clonotypes.

2024-09-07 05:50:43
Statistical analysis and report generation for results.

Analysis Finished
Elapsed Time: 6 hours 58 minutes 25 seconds
```

成功的运行会以Analysis Finished结束。