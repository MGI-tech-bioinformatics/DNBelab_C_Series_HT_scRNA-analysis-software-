# 命令行参数



每个分析的命令行选项分为必须参数，可选参数和标志参数。必须参数和可选需要输入，而标志参数则不需要提供输入。

使用 `dnbc4tools <subtypes> <subcommands> --help` 可获取帮助信息。

</br>
</br>

## [单细胞RNA分析](./scRNA.md)

- **dnbc4tools rna run**

  RNA主分析流程。使用单细胞 RNA 的 cDNA 和 oligo 文库测序数据，进行质量控制、比对和功能区域注释。随后，合并磁珠并识别细胞生成原始基因表达矩阵，识别细胞生成过滤后的基因表达矩阵。接着，对该矩阵进行细胞过滤、降维、聚类和注释分析，最终生成 HTML 格式的网页报告并输出分析结果。

- **dnbc4tools rna mkref**

  构建用于单细胞 RNA 分析的比对和注释参考数据库。

- **dnbc4tools rna multi** 

  多样本操作，每个样本执行单细胞 RNA run 主分析流程。

</br>
</br>

## [单细胞ATAC分析](./scATAC.md)

- **dnbc4tools atac run**

  主分析流程。使用单细胞 ATAC 文库测序数据，经过过滤和比对生成所有磁珠的 fragments 文件。合并磁珠并执行 peak 调用分析，利用 peaks 区域的片段信息进行细胞识别。随后进行细胞过滤、降维和聚类，最终整合各步骤结果生成 HTML 网页报告并输出分析结果。

- **dnbc4tools atac mkref**

  构建用于单细胞 ATAC 分析的 reads 比对参考数据库。

- **dnbc4tools atac multi** 

  多样本操作，每个样本执行单细胞 ATAC run 主分析流程。

</br>
</br> 

## [单细胞VDJ分析](./scVDJ.md)

- **dnbc4tools vdj run**

  主分析流程。使用单细胞 VDJ 文库测序数据和对应样本的 5' 转录组分析结果。首先对数据进行过滤，利用 5' 转录组结果合并磁珠，接着比对 VDJ 基因区域并提取对应的 reads，进行从头组装并注释。根据组装注释结果和 5' 转录组的细胞获取情况进行细胞过滤，最后整合各步骤结果生成 HTML 网页报告并输出分析结果。

</br>
</br>

## 工具类

- **dnbc4tools tools mkgtf**

  GTF 文件操作工具，包括类型统计、基因过滤和文件格式检查。

- **dnbc4tools tools changetag**

  调整 BAM 标签信息的工具。

- **dnbc4tools tools clean**

  清理分析中间文件的工具。