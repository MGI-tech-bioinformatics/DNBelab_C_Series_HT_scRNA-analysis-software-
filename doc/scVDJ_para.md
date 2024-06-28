## DNBelab C4 VDJ Analysis Workflow

### dnbc4tools vdj run

**Required Parameters:**

- **--name**
  - Define the sample name during usage, and the sample ID displayed in the result HTML report will be the same as this name.
- **--fastq1, --fastq2**
  - fastq1 and fastq2 represent the R1 and R2 sequences of the VDJ library.
  - If there are multiple sequences, separate them with commas, ensuring that the corresponding R1 and R2 sequences are consistently sorted. Multiple fastq files require consistent dark reaction settings. Inconsistent dark reactions, resulting in different library lengths, may lead to analysis errors, if not, before analysis, you can use other tools to trim them.
  - Data from different experiments or different samples cannot be merged for analysis. Only data from the same library can be merged for analysis.
- **--ref**
  - The software comes with built-in reference databases for Human and Mouse. You can directly use the database corresponding to the species. Analysis for other species is not supported at this time.
- **--chain**
  - Force the analysis to be carried out for a particular chain type. The accepted values are: “TR” for T cell receptors; “IG” for B cell receptors.


- **--beadstrans**
  - Required parameters. The analysis of the 5' scRNA must be completed first. In the analysis results directory, there should be a file named "singlecell.csv" that records the correspondence between cells and beads.

**Optional Parameters:**

- **--outdir**
  - The directory where the results are saved. The directory name during output is based on the sample ID provided with the `--name` parameter, defaulting to the current directory.
- **--threads**
  - The quantity of threads utilized during the analysis. Increasing the number of threads can accelerate the analysis speed.
- **--darkreaction**
  - The software can automatically identify dark reaction settings based on the R1R2 sequence structure of  library. Dark reaction refers to biochemical reactions without base identification, often set at fixed bases. The logic for identification is as follows: the software first examines the length of the first 200,000 sequences to determine the presence of dark reactions. 
  - Automatic detection is recommended. Dark cycle modes can be "R1R2", "R1", "R2", "unset",  where "R1R2" indicates a dark reaction set for Read1 and Read2.
- **--customize**
  - For specialized requirements beyond standard settings, users can directly use `--customize` modify the corresponding information in a whitelist JSON file. [See the JSON file format for more details](./json.md).
- **--process**
- Set analysis steps, including:
    - **data:** Perform quality control and alignment of library sequencing data to obtain sequences that can be aligned to VDJ gene sequences.
    
  - **assembly:** Use TRUST4 to assemble reads from each cell into contigs and annotate them using the IMGT database.
  
  - **filter:** Perform cell calling based on contig sequence length, whether the sequence is full-length, UMI content, etc. Identify droplets that contain T or B cells. Group cell-associated barcodes into clonotypes.
  
  - **report:** Generate result files and HTML web reports. 

**Flag Parameters:**

- **--singleEnd**
  - Specify the sequencing type of the FASTQ files that need to be analyzed. Determine whether to use only R2 reads or paired-end reads for assembly.