# BIOSCAN QC 
### Version 2.1 [November 2024]
<br><br>
This script generates a BIOSCAN QC report and filters one consensus sequence per sample using output files from mBRAVE. It is designed specifically for processing the Wellcome Sanger Institute BIOSCAN data. <br> <br> 
<p align="center">
  <img src="./2024Sep_QC2.png" alt="QC Repor"/>
</p>

## Installation
No installation is required when running this script on Farm.
## Usage
Submit an interactive job request:<br>
```bash
bsub -Is -n 4 -R "select[mem>2000] rusage[mem=2000] span[hosts=1]" -M 2000 -G team222 bash
```
Set the batch number [the batch number must match the name of the input data directory; replace X with a corresponding number]:<br>
```bash
batch="batchX"
```
Run the script:<br>
```bash
export batch_path="/nfs/users/nfs_a/aw43/aw43/2024_07_bioscan_qc/input/mbrave_batch_data/${batch}/"
export output_path="/nfs/users/nfs_a/aw43/aw43/2024_07_bioscan_qc/input/output/qc_reports/${batch}/"
export batch_no="${batch}"
module load HGI/softpack/users/aw43/aw43_bioscan-aw43_bioscan-4-aw43_bioscan-4/1
Rscript -e "rmarkdown::render(input = '/lustre/scratch126/tol/teams/lawniczak/users/aw43/2024_07_bioscan_qc/code/QCBioscan.Rmd', output_format = 'html_document', output_dir = Sys.getenv('output_path'))"
```
<br>
<p align="center">
  <img src="./2024Sep_QC.png" alt="QC Repor"/>
</p>

## Input
The script requires the following files from mBRAVE:
<i>
<li>sample_stats.txt 
</li>
<li>control_pos_stats.txt
</li>
<li>control_neg_stats.txt
</li>
<li>consensusseq.fas
</li>
<li>consensusseq_network.tsv
</li>
</i>
<br>
The fasta file should contain all sequences, not just the consensus sequences. No filtering should be applied when downloading data from mBRAVE. The names of the downloaded files should not be altered and should contain batch numbers and identifiers. <br><br>
Additionally, the script automatically loads a .csv file with UMI indices from the Farm directory. The directories are set automatically and should not be altered. <br>

## Output
The script generates the following output files:
- <i>QCBioscan.html</i><br>Main QC report with plots, tables, and statistics for the sequencing run.
- <i>filtered_metadata.csv</i><br>Metadata for samples that passed the QC with information for each sample. Only one Arthropod sequence per sample is included. Two metadata files are generated: 1. Contains all samples that passed the QC and their quality scores [see below]; 2. Contains only samples with satisfactory quality scores. Controls are not included. 
- <i>filtered_sequences.fasta</i><br>Consensus sequences for all samples included in the metadata. Two versions are generates: 1. Contains all sequences for samples that passed the QC; 2. Contains only sequences for samples with satisfactory quality scores [ready for BOLD upload].
- <i>read_summary_metadata.csv</i><br>Summary statistics for the sequencing run including sample numbers, control statistics, and quality score distribution. The tables can be combined across the batches to compare sequencing runs and to calculate statistics. 
- <i>unique_secondary_sequences.csv</i><br>Table of secondary sequences not found elsewhere on the partner or UMI plate [5 or more reads], retained for further secondary sequence analysis [parasites/symbionts].
- <i>conflicts.csv </i><br>Tables of conflicting secondary sequences detected across the samples. Sequences with good read support (> 50 reads or 50% or more of the primary sequence read) that were assigned to a different taxon than the primary sequence [family or order level] are flagged in these files for further secondary sequence analysis [parasites/symbionts]. This cut-off is not used to exclude any samples. Rather, it is used to detect candidate samples that may have two different arthropods in them or have biologically relevant signals of two arthropods interacting. 
- <i>tardigrada_nematoda_rotifera_annelida.csv and wolbachia.csv</i><br>Non-Arthropod sequences and metadata retained for further exploration. These files are not filtered for number of reads nor contain quality categories. These should be processed further if required. 

## Documentation
The QC process is divided into parts:<br>
1. <i>Assessment of the sequencing run</i><br>
2. <i>Assessment of sequence conflicts and contaminants</i><br>
3. <i>Final assessments and plots </i><br><br>

<i>Assessment of the sequencing run</i><br><br>
The script evaluates the quality of the sequencing run by providing statistics for both control and sample data, and quality assessments of the partner and UMI plates.<br>

Statistics for the positive controls <br>
Positive controls are always in G12
- Total number of positive conrols per batch and per plate.
- Total number of reads in positive controls.
- Maximum and minimum number of reads in positive controls and the associated positive control samples.
- Median, average, standard deviation, and quantiles calculated for the positive control reads in a given batch.
- Histogram showing the read count distribution across positive controls 
- Number and names of positive controls in the lower 5% quantile and the names of associated partners.

Statistics for the negative controls <br>
<b>Lysate controls:</b> H12 and any wells left empty by the partners on partially empty plates <br> 
<b>Empty control:</b> one well per plate selected at random [insect automatically excluded]
- Total number of negative controls. Numbers of empty and lysate negative controls in a given batch.
- Number of negative controls per plate.
- Total number of reads in empty and lysate negative controls.
- Minimum and maximum number of reads in empty and lysate negative controls.
- Number of empty and lysate negative controls with 0 reads.
- Median, average, skewness, and quantiles calculated for the empty and lysate negative controls.
- Histograms showing the read count distribution across empty, lysate, and all negative controls.
- Number of empty and lysate negative controls in the upper 5% and 3% quantiles and the names of associated partners.

Statistics for the samples <br>
- Total number of samples in the batch.
- Total numger of partner plates in the batch.
- Total number of sample reads.
- Maximum and minimum number of sample reads [including number of samples with 0 reads if present].
- Median, average, standard deviation, skewness, and quantiles.
- Histogram showing the read count distribution for all samples.
- Number of samples in the lower 10% and 5% quantiles and the names of associated partners.
- Number of samples with 0 reads.

Partner plate boxplot <br>
The plot shows the total read count per sample for each partner plate [each box]. Box colours indicate partners. The grey horisontal line shows the median, while the brown horisontal like indicates the mean. Blue data points show empty negative controls. Navy data points show lysate negative controls. Green data points show positive controls. 
Boxes that are grey inside show plates where the 75th percentile of the data is lower than expected mean read count. These plates and flagged as 'low-performance' need further attention. 

UMI plate boxplot <br> 
The plot shows the read count per sample for each UMI plate [each box]. Data point colours indicate partners. The grey horisontal line shows the median, while the brown horisontal like indicates the mean. Blue data points show empty negative controls. Navy data points show lysate negative controls. Green data points show positive controls. Boxes that are grey inside show plates where the 75th percentile of the data is lower than expected mean read count.
Purple data points transfer the information from the partner plate boxplot. Namely, all purple data points are samples from the 'low-performance' partner plates. The percentage of samples from the low-performance partner plates that are present in the low-performance UMI plates is displayed below the plot. Usually, the 'low-performance' of UMI plates is caused by accumulation of samples from 'low-performance' partner plates, therefore, it is a carry-on effect and not the failure of UMI plates.  

Plates flagged as 'low-performance' are not automatically eliminated. Highlighted plates should be examined and re-sequences if required.

Further, read counts in positive controls in the lower 5% quantile are compared to the average read counts in the samples from the corresponding plate to assess whether the negative control failed or the entire plate has low read count. 

Further, all 'low-performance' partner plates are displayed as heatmaps to facilitate manual evaluation process. 
<br><br><br>
<i>Assessment of sequence conflicts and contaminants</i><br><br>
First, the script will merge the mBRAVE consensusseq_network.tsv and consensusseq.fas. An error message is displayed if any of the sequences is not matched to the corresponding sample record. 

Positive control as contamination source <br>
- List of non-positve control samples that contain positive control reads [based on the positive control OTU].
- Number of samples with positive control reads as primary or secondary sequence.
- Heatmap plot showing the location of the contaminants relative to the source for both the partner and UMI plates. Colour of the squares reflect the number of reads. Negative controls are outlined, colour of the outline correspond with partners. Squares that are not outlined represent potential sources of contamination within plates: identical sequences found within these wells and negative controls. Negative controls with within the higher 2% quantile of reads have thicker chartreuse outline.<br><b>NOTE:</b> No Bovidae reads are insluded at this step. The current setup allows for a change of 2 nucleotides in the sequence - the sequences found in samples and the negative controls may not be 100% identical [minimum 99.695%]. 
- Histogram of read counts of all secondary sequences across all samples. It is here to assess whether the contamination level in samples that contain secondary positive control sequences is comarable to the contamination level in samples that contain other unidentified sequences [potential micro sphashes].
Usually, the number of secondary sequence reads resulting from random splashes is higher than the number of contamination reads from positive control samples. However, the lines should be in close proximity.
- At this point, if there's a sample with positive control as a primary sequence, it is going to be replaced with the best [highest read count and similarity score] secondary arthropod sequence or excluded if there's not suitable sequence replacement. 


Negative control contamination<br>
- Distribution of read counts in negative controls - primary and secondary sequences. At this point negative controls are pooled together and no differentiation between empty and lysate controls is applied. Contamination source can be either primary or secondary contig within samples!
- A table of families that are the most common sources of contamination in negative controls.
- Two heatmaps [for partner and UMI plates] showing contamination levels and potential sources.
- The current script allows for a change of 2 nucleotides in the sequence - the sequences found in negative controls and potential corresponding sources of contamination within a plate may not be 100% identical [minimum 99.695% identity].


Assessment of primary and secondary sequences<br>
- Number of samples with a primary sequence only.
- Number of samples with primary and secondary sequences.
- Number and percentage of chimeric primary sequences.
- Number and percentage of chimeric secondary sequences.
- Number and percentage of EXCLUDED primary sequences.
- Number and percentage of primary sequences with no taxonomy assigned.

The next step is evaluation of samples with no taxonomy assigned. The first plot shows three histograms of sequence length distribution: I. All sequences; II. A random sequence subset; III. All samples with no taxonomy assigned. These plots were used to evaluate whether shorter sequences systematically do not get mBRAVE taxonomy assigned to them. The script identifies sequences shorter than expected within those without assigned taxonomy and replaces them with the closest matching longer sequence within the well [on average 100 bp longer; Levenshtein distance < 150].

The next step is evaluation of samples where the primary sequence is not an arthropod. Visual examination of over 50 photos revealed that when the reads come primarly from Bovidae, Nematoda, Annelida, Wolbachia, Rotifera, Tardigrada or human, the best [highest read count and similarity score] secondary sequence comes from the arthropod plated in the well, unless any other errors, including sequencing erros occur. Therefore, the primary non-arthropod sequences get removed and replaced by the best secondary sequence. The report displays number of primary non-arthopod sequences in a batch by phylum. Samples where exclusively non-arthropod sequences were detected get removed at this step. At this point Nematoda, Tardigrada, Rotifera, Annelida, and Wolbachia sequences and sample information gets saved to seperate output files. 

The next step is detection of Anopheles mosquitoes [BOLD:AAA3436] that BIOSCAN uses as internal controls. These are going to be detected and evaluated only if campus samples are present in the batch. Samples with primary Anopheles sequence with 250 or more reads get counted and removed. Samples where Anopheles sequences are secondary are removed from further steps.

The next step is evaluation of secondary arthropod sequences in the remaining samples. The report displays how many samples have no conflicts at all, meaning that in a given well only one sequence is present. Next, all sesuences [primary and secondary] get removed if they have less than 5 reads. The number of excluded and retained samples is displayed in a table. Fewer than 5 reads cannot confidently support the sample. This cut-off was established based on the manual examination of 30 photos per read count per sample [Figure 1]. Photos were divided into two categories: correct family-level taxonomy and incorrect family-level taxonomy. 

<p align="center">
  <img src="./taxonomy_family_level2.png" alt="QC Repor"/>
</p>
<b>Figure 1. A.</b> Photo examination and identification to the family level indicate that familt-level taxonomic identification by mBRAVE is more accurate for samples with higher read support. Specifically, when samples have only one read, the majority of specimens are assigned incorrect family-level taxonomy. In samples with five or more reads, the error rate is around 10%. This error rate is significantly reduced when the read count reaches approximately 50 reads for the consensus sequence. These cut-offs (5 and 50 reads) were applied to sample quality scores to exclude samples with insufficient read support (fewer than 5 reads) and those with conflicting sequences (fewer than 50 reads).
<b>B.</b> Breakdown of the number of samples from 23 sequencing runs categorized by read count. The majority of samples have over 200 reads supporting the consensus sequence. The likelihood of accurate taxonomy assignment does not change significantly across categories, except in cases where samples with more than 100 reads have fewer instances of no taxonomy assigned. It is crucial to note that the taxonomy assigned to samples by mBRAVE may still be incorrect, as shown in the first plot.
<br><br>
<p align="center">
  <img src="./contaminant_level.png" alt="QC Repor"/>
</p>
<b>Figure 2. A.</b> Eight <i>Anopheles</i> sp. individuals were randomly added to the Campus Plates (Wellcome Sanger Institute). The plot represents the read count of these mosquitoes detected in other samples as secondary (contamination) reads. In 99% of cases, the read support for these secondary reads was approximately 52 reads or lower. Therefore, we apply a cut-off of 50 reads to support primary sequences in samples with any secondary reads detected, as the detected sequence may represent contamination rather than the actual insect.
<b>B.</b> Distribution of read support in the empty negative controls. Empty negative controls are randomly selected by excluding an insect from each plate. In 95% of cases, the primary sequences detected in these controls had around 30 reads supporting them, which is lower than the 50-read threshold. However, we did observe read counts as high as 320, indicating that cross-plate contamination, though rare, can occur at relatively high levels. In these cases, the primary sequence from the sampled insect may "compete" with the contaminant sequence. Based on this information, we temporarily exclude any samples with secondary sequences that have more than 5 reads of support and retain them for further conflict assessment.
<br><br>

Next, the script identifies secondary sequences that are not found elsewhere on the same partner or UMI plate where the sample was processed. These sequences, along with the corresponding sample information, are saved to a separate file for further evaluation. Similarly, information about secondary sequences that conflict with the primary sequence is saved to another separate file. Conflicts are defined as cases where two sequences within a sample have different family or order classifications. In the previous version, only conflicting sequences with read support of more than 50 reads were retained if the primary sequence was supported by 100 or more reads or had more than 50% of the primary sequence's read count. Currently, all conflicts are saved for further evaluation, and samples that meet the specified read count criteria are flagged within the output file.

In the next step, all secondary sequences are removed, leaving only the primary arthropod sequences [or primary sequences with no taxonomy assigned]. The report then displays the number and percentage of retained samples. 

At this step, all retained samples get assigned a quality score: 

| Score | No. reads in primary | Sample description [secondary sequence assessment]                               | Decision     |
|-------|----------------------|--------------------------------------------------------------|-----------------------------------------|
| <b>1</b> | > 200   | Only one sequence with more than 200 reads, no secondary sequence detected                             |YES |
| <b>2</b> | 50 - 200| Only one sequence with 50 to 200 reads, no secondary sequence detected                                 |YES |
| <b>3</b> | 5 - 49  | Only one sequence with 5 or more but less than 50 reads, no secondary sequence detected                |YES |
| <b>4</b> | > 200   | Dominant sequence with more than 200 reads, non-conflicting secondary sequences with 5 or less reads   |YES |
| <b>5</b> | 50 - 200| Dominant sequence with 50 to 200 reads, non-conflicting secondary sequences with 5 or less reads       |YES |
| <b>6</b> | > 200   | Dominant sequence with more than 200 reads, conflicting secondary sequences with 5 or less reads       |YES |
| <b>7</b> | 50 - 200| Dominant sequence with 50 to 200 reads, conflicting secondary sequences with 5 or less read            |YES |
| <b>8</b> | > 200   | Dominant sequence with more than 200 reads, secondary sequences with more than 5 read support          |NO  |
| <b>9</b> | 50 - 200| Dominant sequence with 50 to 200 reads, secondary sequences with more than 5 read support              |NO  |
| <b>10</b>| 6 - 49  | Dominant sequence with 5 or more but less than 50 reads, non-conflicting secondary sequences with less than 5 reads |NO  |
| <b>11</b>| 6 - 49  | Dominant sequence with more than 5 but less than 50 reads, any other secondary reads present<br>[conflicting or not]|NO  | 

The report displays the above table with the number of samples assigned to each category. The categories are also included in the final sample metadata. The Decision column indicates whether the samples assigned these categories are going to be included in the BOLD upload. 

<p align="center">
  <img src="./qc-catogory-boxplot.png" alt="QC Repor"/>
</p>
<b>Figure 3.</b> Number of samples assigned to each of the quality scores within 20 sequencing batches. The majority of samples are of good quality and do not contain conflicting nor secondary sequences. Samples within categories 8 to 11 will be examined further. 
<br><br>
<p align="center">
  <img src="./taxonomy_bias.png" alt="QC Repor"/>
</p>
<b>Figure 4.</b> Evaluation of order-level taxonomic assignment to samples based on their read support for the primary consensus sequence. The plot on the right shows the breakdown of small insect orders. The distribution of orders is not associated with read support, indicating that no specific orders are more likely to be poorly sequenced or excluded due to the applied quality control cut-offs, thereby causing potential biases in the data. The only order found in categories that could potentially be excluded if secondary sequences are present is Megaloptera. However, as only a few individuals from this order were detected, further evaluation is necessary to assess any potential biases.
<br><br>
<i>Final assessments and plots </i><br><br>
This part contains:
- Big heatmaps showing the number or sequenced reads per plate well.
- Two similar heatmaps showing the read support for the consesnus sequence. The first heatmap displays all retained consensus sequences, the second heatmap shows only the samples in which the decision category indicates they will be retained for BOLD upload.
- Histogram showing the sequence length distribution across the retained samples. 
- Tables with percentages of retained samples per partner, partner plate, and UMI plate. 
- All partner plates and UMI plates displayed as heatmaps.
