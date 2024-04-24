## DNBelab C4 RNA Analysis Workflow



### dnbc4tools rna run

**Required Parameters:**

- **--name**
  - Define the sample name for usage, ensuring consistency with the sample ID displayed in the resulting HTML report.
- **--cDNAfastq1, --cDNAfastq2, --oligofastq1, --oligofastq2**
  - cDNAfastq1 and cDNAfastq2 represent the R1 and R2 sequences of the cDNA library,  oligofastq1 and oligofastq2 represent the R1 and R2 sequences of the oligo library.
  - Separate multiple sequences with commas, and ensure consistent sorting of corresponding R1 and R2 sequences. It's essential to have consistent sequencing modes for the fastq files, meaning consistent dark reaction settings.
  - Data from different experiments or different samples cannot be merged for analysis. Only data from the same library can be merged for analysis.
- **--genomeDir**
  - Specify the directory containing the alignment database generated during single-cell RNA library preparation. This includes genome files, GTF format annotation files, STAR alignment database (version 2.7.2b), mitochondrial chromosome names, and a list of mitochondrial genes.

**Optional Parameters:**

- **--outdir**:

  - Designate the directory for saving results. The directory name will be based on the sample ID provided with the `--name` parameter, defaulting to the current directory.

- **--threads**:

  - Set the number of threads used during analysis to accelerate processing speed.

- **--calling_method, --expectcells, --forcecells**:

  - `--calling_method`  is the method used to identify true cells during analysis. Two options are available: "[emptydrops](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-019-1662-y)" and "barcoderanks." . By default, the "emptydrops" method is selected. 

  - In the "emptydrops" algorithm, cells are initially identified using the expected number of cells parameter(`--expectcells`) to capture regions with high UMIs (unique molecular identifiers). Cells with UMIs below the threshold (default is 1000, adjustable with the `--minumi` parameter) are filtered out. Finally, cells with significant differences compared to the background expression matrix are included as true cells.

  - The "barcoderanks" method determines genuine cells based on the sorting curve of UMIs, considering the inflection point of the curve as the threshold. Cells with UMIs higher than this point are classified as true cells.

  - `--forcecells` option extracts a specified number of cells based on UMI sorting to determine true cells.

- **--chemistry, --darkreaction, --customize**:

  - The software can automatically identify reagent types and dark reaction settings based on the R1R2 sequence structure of cDNA and oligo libraries. Dark reaction refers to biochemical reactions without base identification, often set at fixed bases. The logic for identification is as follows: the software first examines the length of the first 200,000 sequences to determine the presence of dark reactions. If no dark reaction is detected, it then uses the fixed base sequence information and positions corresponding to different reagent versions to identify the appropriate whitelist structure file.

  - If automatic identification fails or is incorrect, manual settings can be applied using `--chemistry` and `--darkreaction`. The `--chemistry` option includes "scRNAv1HT", "scRNAv2HT", and "scRNAv3HT", while the `--darkreaction` option uses commas to separate cDNA and oligo settings, such as "R1,R1R2", "R1,R1", "unset,unset", etc., where "R1,R1R2" indicates a dark reaction set for cDNA R1 and oligo R1R2.

  - For specialized requirements beyond standard settings, users can directly use `--customize` modify the corresponding information in a whitelist JSON file. [See the JSON file format for more details](./json.md).

- **--process**:

  - Set analysis steps, including:
  
    - **data**: Perform QC and alignment for cDNA library data, determine alignment positions based on annotation information, and generate *final_sorted.bam* files. Also, perform QC for oligo library data and generate *CB_UB_count.txt* files based on R2-end information.
    - **count**: Calculate the similarity between cell barcodes based on cDNA and oligo information, merge cell barcodes with high similarity into the same cell, generate the merged raw expression matrix, identify true cells using cell calling, generate filtered expression matrix, and perform saturation calculation.
    - **analysis**: Filter cells and perform dimensionality reduction, clustering, and annotation on the filtered expression matrix.
    - **report**: Generate result files and HTML web reports.
  
  - Choose the analysis step to skip completed steps.
  
    - Among all parameters, `--calling_method`,` --expectcells`,and `--forcecells` occur during the *count* step. Adjusting these parameters may skip the data step. Use *--process count, analysis, report.*
  
    - Parameters `--chemistry`, `--darkreaction`,` --customize`,` --no_introns`, and `--end5` occur during the *data* step. If adjustments are needed, the process cannot be skipped. Use the entire process by default: *--process data, count, analysis, report*.

**Flag Parameters:**

- **--no_introns**
  - Filter out reads from intronic regions during analysis, retaining only reads from exon regions for expression quantification.
- **--end5**
  - Analyze 5' end transcriptome data.


</br>
</br>
</br>

### dnbc4tools rna mkref

**Required**:

- **--fasta, --ingtf**
  - The reference genome FASTA and GTF annotation files for your species.
  - If available in the Ensembl database, we recommend using files from there. If your species of interest is not available in Ensembl, GTF and FASTA files from other sources can also be used. Note that a GTF file is required, but GFF files are not supported. The genome FASTA file is recommended to be the primary assembly.
  - Format requirements for a GTF file: For Single-Cell RNA analysis, GTF file must include at least "gene" or "transcript" types and "exon" type annotations. Attributes should have at least one of "gene_id" or "gene_name" and one of "transcript_id" or "transcript_name".

**Optional**:

- **--species**
  - Specify the species name for constructing the reference database. For cell annotation analysis, only "Homo_sapiens", "Human", "Mus_musculus", and "Mouse" are valid options.
- **--chrM**
  - Identify mitochondrial chromosomes. "Auto" recognizes "chrM, MT, chrMT, mt, Mt". Genes located on mitochondria will be automatically identified, generating the file "mtgene.list".
- **--genomeDir**
  - Path to the directory where database files will be stored.
- **--limitram**
  - Maximum available RAM (bytes) for genome index generation.
- **--threads**
  - The quantity of threads utilized during the analysis. Increasing the number of threads can accelerate the analysis speed.

**Flag Parameters**:

- **--noindex**
  - Skip indexing step if the database has already been constructed using scSTAR.



For genomes with numerous and varying chromosome sizes, the library construction has been adapted to automatically determine the values of `genomeSAindexNbases` and `genomeChrBinNbits`. Upon completion of the library construction, a `ref.json` file is generated in the database directory to record key information.

```json
{
    "species": "Homo_sapiens",
    "genome": "$PATH/genome.fa",
    "gtf": "$PATH/genes.filter.gtf",
    "genomeDir": "$PATH",
    "chrmt": "chrM",
    "mtgenes": "$PATH/mtgene.list"
}
```

</br>
</br>
</br>


###  dnbc4tools rna multi

**Required**:

- **--list**

  - Generate a three-column list with tab (\t) separators. The first column contains sample names, the second contains cDNA library sequencing data, and the third contains oligo library sequencing data. Multiple fastq files should be separated by commas, and R1 and R2 files should be separated by semicolons.

  - Here's an example of how the input list should be formatted: 
```
$sample1 cDNA1_R1.fq.gz;cDNA1_R2.fq.gz oligo1_R1.fq.gz,oligo4_R1.fq.gz;oligo1_R2.fq.gz,oligo4_R2.fq.gz 
$sample2 cDNA2_R1.fq.gz;cDNA2_R2.fq.gz oligo2_R1.fq.gz;oligo2_R2.fq.gz 
$sample3 cDNA3_R1.fq.gz;cDNA3_R2.fq.gz oligo3_R1.fq.gz;oligo3_R2.fq.gz
```

**Other Parameters**: Refer to the **run** section.
