## DNBelab C4 ATAC Analysis Workflow

### dnbc4tools atac run

**Required Parameters:**

- **--name**
  - Define the sample name during usage, and the sample ID displayed in the result HTML report will be the same as this name.
- **--fastq1, --fastq2**
  - fastq1 and fastq2 represent the R1 and R2 sequences of theATAC library.
  - If there are multiple sequences, separate them with commas, ensuring that the corresponding R1 and R2 sequences are consistently sorted. Multiple fastq files require consistent dark reaction settings. Inconsistent dark reactions, resulting in different library lengths, may lead to analysis errors, if not, before analysis, you can use other tools to trim them.
  - Data from different experiments or different samples cannot be merged for analysis. Only data from the same library can be merged for analysis.
- **--genomeDir**
  - The genomeDir refers to the alignment database generated during single-cell ATAC library preparation, containing the genome files, bed format transcript start site files, chromap alignment database , mitochondrial chromosome names, and chromosome and genome size.

**Optional Parameters:**

- **--outdir**
  - The directory where the results are saved. The directory name during output is based on the sample ID provided with the `--name` parameter, defaulting to the current directory.

- **--threads**
  - The quantity of threads utilized during the analysis. Increasing the number of threads can accelerate the analysis speed.

- **--darkreaction**
  - The software can automatically identify dark reaction settings based on the R1R2 sequence structure of  library. Dark reaction refers to biochemical reactions without base identification, often set at fixed bases. The logic for identification is as follows: the software first examines the length of the first 200,000 sequences to determine the presence of dark reactions. 
  - Automatic detection is recommended. Dark cycle modes can be "R1R2", "R1", "R2", "unset",  where "R1R2" indicates a dark reaction set for Read1 and Read2.

- **--customize**
  - Comma-separated string value: [r1|r2|bc]\:start\:end:strand. Inclusive start and end, -1 means end of read. Example: "bc:6:15,bc:22:31,r1:65:-1,r2:19:-1". "bc" indicates cell barcode information, "6:15" denotes positions 7 to 16 in the sequence. The numerical values for positions start from 0, differing by 1 from the actual positions.

- **--forcecells**
  - Extracts a specified number of cells based on fragments overlapping peaks numbers  sorting to determine true cells. This value has the highest level of authority.

- **--frags_cutoff, --tss_cutoff, --merge_cutoff**
  - `frags_cutoff` is the minimum number of fragments used for cell filtering during the cell calling stage. It defaults to 1000. `merge_cutoff` is the minimum number of fragments required for cell barcode merging; it defaults to 1000. In the peak calling step, only cells with fragment counts exceeding `merge_cutoff` are considered. It's recommended to keep `merge_cutoff` and `frags_cutoff` consistent or lower than this value. 
  - `tss_cutoff` refers to the fraction of fragments overlapping transcription start sites (TSS) below which cells are filtered out, and it defaults to no filtering. 
  - It's advisable to use the default parameters for analysis and adjust these values after obtaining the result report.

- **--process**

  - Set analysis steps, including:
    - **Data:** Perform quality control and alignment of library sequencing data. Generate the raw fragments.tsv.gz file and filter it to retain only the chromosome information specified in the chromeSize file. Compute Jaccard distance for cell barcodes with fragment counts exceeding the merge_cutoff and plot the curve. Use the Otsu algorithm to obtain the merged trimming value. Merge cell barcodes and generate the merged all.merge.fragments.tsv.gz file. Perform peak calling analysis to obtain peak position information.

    - **Decon:** Utilize peak position information and fragments data to generate the raw_peak_matrix. Identify true cells (filter based on fragments overlapping peaks curve, fragments number, and fraction of fragments overlapping transcription start sites). Generate the filter_peak_matrix. Compute TSS enrichment score and perform saturation analysis.

    - **Analysis:** Filter cells and perform dimensionality reduction clustering.

    - **Report:** Generate result files and HTML web reports. Choose the analysis step to skip completed steps.

  - Among all parameters, `---forcecells`, `--frags_cutoff`, and `--tss_cutoff` occur during the decon step. Adjusting these parameters may skip the data step. Use *--process decon, analysis, report.* Parameters `--darkreaction`, `--customize`, `--merge_cutoff`, and `--bam` occur during the data step. If adjustments are needed, the process cannot be skipped. Use the entire process by default: *--process data, decon, analysis, report*.

**Flag Parameters:**

- **--bam**
  - Generating the BAM file during alignment significantly increases the software analysis time. It is not recommended to use this parameter if BAM files are not needed.



</br>
</br>

### dnbc4tools atac mkref

**Required Parameters:**

- **--fasta, --ingtf**
  - The reference genome FASTA and GTF annotation files for your species. If available in the Ensembl database, we recommend using files from there. If your species of interest is not available in Ensembl, GTF and FASTA files from other sources can also be used. Note that a GTF file is required, but GFF files are not supported. The genome FASTA file is recommended to be the primary assembly.

**Optional Parameters:**

- **--genomeDir**
  - Path to the directory where database files will be stored.
- **--species**
  - Specify the species name for constructing the reference database.
- **--tag**
  - When generating the tss.bed file, use gene information or transcript information. The default is to use transcript.
- **--chrM**
  - Identify mitochondrial chromosomes. "Auto" recognizes "chrM, MT, chrMT, mt, Mt".
- **--chloroplast**
  - Set chloroplast chromosome names. It is recommended to set chromosome names for plant samples. When the number of fragments from mitochondria and chloroplast is extremely high, it may lead to excessive memory consumption during the merging beads step and cause errors, as well as artificially high TSS proportions.
- **--prefix**
  - String or list of strings representing prefix(es) or full name(s) of chromosomes to keep. For example, "--prefix chr" selects chromosome sequences starting with "chr", or "--prefix 1,2,3,4,5,Mt,Pt" selects chromosome sequences of Arabidopsis thaliana.

**Flag Parameters:**

- **--noindex**
  - Skip indexing step if the database has been built using chromap. Only generate ref.json file.



The [issue](https://github.com/haowenz/chromap/issues/131) with using Chromap for library construction and alignment is its inability to handle extremely large genomes. As a result, some species cannot utilize this software for scATAC analysis. Future updates will explore alternative alignment methods for super-sized genomes. Upon completion of the library construction, a `ref.json` file is generated in the database directory to record key information.

```
{
    "species": "Homo_sapiens",
    "genome": "genome.fa",
    "index": "genome.index",
    "chromeSize": "chrom.sizes",
    "tss": "tss.bed",
    "promoter": "promoter.bed",
    "blacklist": "None",
    "chrmt": "chrM",
    "chloroplast": "None",
    "genomesize": "hs"
}
```

- The chromosome names listed in the `chromeSize` file will be included in the `fragments.tsv.gz` file for analysis. Chromosomes not listed in this file will be excluded from the analysis. 

- In version 2.1.2, the `blacklist` parameter has been removed, and the presence of a blacklist file is no longer mandatory. If required, it can be manually added. This parameter does not affect the analysis results. The number of fragments in blacklist regions will be recorded in the metadata file, `singlecell.csv`, under the column `blacklist_region_fragments`. 
- The `genomesize` value is used for MACS2 peak calling analysis. MACS2 has special identifiers for certain species, such as "hs" for Homo sapiens.

</br>
</br>

###  dnbc4tools atac multi

**Required**:

- **--list**
  - Generate a two-column list with tab (\t) separators. The first column contains sample names, the second contains library sequencing data. Multiple fastq files should be separated by commas, and R1 and R2 files should be separated by semicolons.

  - Here's an example of how the input list should be formatted: 

```
sample1 test1_R1.fq.gz,test4_R1.fq.gz;test1_R2.fq.gz,test4_R2.fq.gz 
sample2 test2_R1.fq.gz;test2_R2.fq.gz 
sample3 test3_R1.fq.gz;test3_R2.fq.gz
```

**Other Parameters**: Refer to the **run** section.
