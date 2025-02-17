Viivi Halla-aho's documentation of the preprocessing steps

`$*name*` refer to variable names which are used when running these commands in a bash script. Steps 7 and 8 in preprocessing are not necessary before the analysis steps, but they are still good ideas to perform them as they might give information that might lead into removing some of the samples from analysis.


1. Quality control for fastq-files using FastQC.
```sh
fastqc $file.fastq.gz –-outdir=$outputdir
```
* Parameters:
    * `--outdir`: specify output directory
* Version: FastQC 0.11.5


2. Trimming the reads using TrimGalore.
```sh
trim_galore –-rrbs –-paired –-fastqc $file1.fastq.gz $file2.fastq.gz –-output_dir=$outputdir
```
* Parameters:
    * `--rrbs`: specifies that input is RRBS sample
    * `--paired`: performs trimming for paired-end files
    * `–-fastqc`: runs FastQC in the default mode after trimming is complete
* Version: TrimGalore 0.4.3


3. The genome files have to be prepared for alignment tools (has to be done only once per reference genome). New alignment done with both hg19 and lambda phage genomes in `$genomefolder` so that they are prepared together and can be used in the alignment step.
```sh
bismark_genome_preparation –-path_to_bowtie $bowtiepath $genomefolder
```
* Parameters:
    * `--path_to_bowtie`: full path to Bowtie2 installation
* Version: Bismark 0.17.0


4. Alignment to the reference genome using Bowtie2 (inside Bismark).
```sh
bismark –-bowtie2 –o $outputfolder $genomefolder -1 $file1.fq.gz -2 $file2.fq.gz
```
*  Parameters:
    * `--bowtie2`: uses Bowtie2 instead of Bowtie1, default setting
    * `-o`: output directory
    * `-1`: #1 mates of paired-end reads
    * `-2`: #2 mates of paired-end reads
*  Version: Bowtie2 2.3.1, Bismark 0.17.0


5. Extracting the context-dependent methylation using Bismark extraction.
```sh
bismark_methylation_extractor –-report -–paired-end –-bedGraph –o $outputfolder –-counts $file.bam
```
This was repeated with parameters `--ignore_r2 3 --ignore_3prime_r2 1` (in addition to the above-mentioned ones) after checking the M bias reports.
* Parameters:
    * `--report`: prints out a short methylation summary
    * `--paired-end`: input files are Bismark result files generated from paired-end data
    * `--bedGraph`: methylation output is written into a sorted bedGraph file that reports the positions of the cytosines and their methylation state
    * `-o`: output directory
    * `--counts`: the methylated and non-methylated counts are included in the output format instead of simple methylation percentage
* Version: Bismark 0.22.3


6. Generating a cytosine methylation report, merging information from both strands for CpGs.
```sh
coverage2cytosine –-merge_CpG –-genome_folder $genomefolder –o $outputfolder $file.bismark.cov
```
* Parameters:
    * `--merge_CpG`: post-processing the genome-wide report to write out an additional coverage file which has the top and bottom strand methylation evidence pooled into single CpG dinucleotide entity
    * `--genome_folder`: providing the folder where reference genome is being stored
    * `-o`: output filename
* Version: Bismark 0.22.3


7. Generates a graphical HTML report page using Bismark alignment report.
```sh
bismark2report –-alignment_report $file_PE_report.txt –-splitting_report $file_splitting_report.txt –-mbias_report $file.M-bias.txt
```
* Parameters:
    * `--alignment_report`: mapping report input
    * `--splitting_report`: splitting report input
    * `--mbias_report`: M-bias report input)
* Version: Bismark 0.22.3


8. Bisulfite conversion efficiency analysis is done using the reads mapped to Lambda phage genome. First the methylation level for these reads is calculated by running `bismark_methylation_extractor` (see step 5), which produces a `PE_report.txt` file where the methylation level is stated and then the level is subtracted from 1 to get the conversion rate.
* Parameters: Same as in step 5.
* Version: Bismark 0.22.3
