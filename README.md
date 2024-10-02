# BIOSCAN QC
This script generates a BIOSCAN QC report and filters one consensus sequence per sample using output files from mBRAVE. It is designed specifically for processing the Wellcome Sanger Institute BIOSCAN data. <br> <br> 
## Installation
No installation is required when running this script on Farm.
## Usage
Submit an interactive job request:<br>
```bash
bsub -Is -n 4 -R "select[mem>2000] rusage[mem=2000] span[hosts=1]" -M 2000 -G team222 bash
```
Set the batch number [the batch number must match the name of the input data directory; replace X with number]:<br>
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
The script requires the following files from the mBRAVE batch output:
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
The fasta file should contain all sequences, not just the consensus sequences. No filtering should be applied when downloading data from mBRAVE. The names of the downloaded files should not be altered and should contain the batch number. <br><br>
Additionally, the script automatically loads a .csv file with UMI indices from the Farm directory. <br>

## Output
The script generates the following output files:
- <i>QCBioscan.html</i><br>Main QC report with plots, tables, and statistics for the sequencing run.
- <i>filtered_metadata.csv</i><br>Metadata for samples that passed the QC with information for each sample. Only one Arthropod sequence information per samples is included. This file also contains the quality scores that each sample gets assigned [see below]. Controls are not included. 
- <i>filtered_sequences.fasta</i><br>Consensus sequences for all samples included in the metadata, no matter the quality scores. This file must be filtered before submitting to BOLD. 
- <i>read_summary_metadata.csv</i><br>Summary statistics for the sequencing run including sample numbers, control statistics, and quality scores. Further the tables can be combined across the batches to compare sequencing runs and to calculate statistics. 
- <i>unique_secondary_sequences.csv</i><br>Table of secondary sequences not found elsewhere on the partner or UMI plate [5 or more reads], retained for further secondary sequence analysis [parasites/symbionts].
- <i>conflicts_family.csv and conflicts_order.csv</i><br>Tables of secondary sequences with good read support (> 50 reads or 50% or more of the primary sequence read) that were assigned to a different taxon than the primary sequence [family or order level], for further secondary sequence analysis [parasites/symbionts]. This cut-off is not used to exclude any samples. Rather, it is used to detect candidate samples that may have two different arthropods in them or have biologically relevant signals of two arthropods interacting. 
- <i>tardigrada_nematoda_rotifera_annelida.csv and wolbachia.csv</i><br>Non-Arthropod sequences retained for further exploration. These files are not filtered for number of reads nor contain quality categories. These should be processed further if required. 

## Documentation
The QC process is divided into parts:<br>
1. <i>Assessment of the sequencing run</i><br>
2. <i>Assessment of sequence conflicts and contaminants</i><br>
3. <i>Final assessments and plots </i><br><br>

<i>Assessment of the sequencing run</i><br><br>
The script evaluates the quality of the sequencing run by providing statistics for both control and sample data and quality assessments of the partner and UMI plates.<br>

Statistics for the positive controls <br>
- Total number of positive conrols per batch and per plate.
- Total number of reads in positive controls.
- Maximum and minimum number of reads in positive controls and the associated positive control samples.
- Median, average, standard deviation, and quantiles calculated for the positive control reads in a given batch.
- Histogram showing the read count distribution across positive controls 
- Number and names of positive controls in the lower 5% quantile and the names of associated partners.

Statistics for the negative controls <br>
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

Partner plate boxplot
The plot shows the read count per sample for each partner plate [each box]. Box colours indicate partners. The grey horisontal line shows the median, while the brown horisontal like indicates the mean. Blue data points show empty negative controls. Navy data points show lysate negative controls. Green data points show positive controls. 
Boxes that are grey inside show plates where the 75th percentile of the data is lower than expected mean read count. These plates and flagged as 'low-performance' need further attention. 

UMI plate boxplot
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
- Plot showing the location of the contaminants relative to the source for both the partner and UMI plates.
- Histogram of read counts of all secondary sequences across all samples. It is here to assess whether the contamination level in samples that contain secondary positive control sequences is comarable to the contamination level in samples that contain other unidentified sequences [potential micro sphashes].
Usually, the number of secondary sequence reads resulting from random splashes is higher than the number of contamination reads from positive control samples. However, the lines should be in close proximity.
- At this point, if there's a sample with positive control as a primary sequence, it is going to be replaced with the best [highest read count and similarity score] secondary arthropod sequence.


Negative control contamination<br>
- Distribution of read counts in negative controls - primary and secondary sequences. At this point negative controls are pooled together and no differentiation between empty and lysate controls is applied. Contamination source can be either primary or secondary contig within samples!
- A table of families that are the most common sources of contamination in negative controls.
- Two heatmaps [for partner and UMI plates] showing contamination levels and potential sources.
- The current script allows for a change of 2 nucleotides in the sequence - the sequences found in negative controls and potential corresponding sources of contamination within a plate may not be 100% identical [minimum 99.695% identity].


Assessment of primary and secondary sequences<br>
- Number of samples with a primary sequence only.
- Number of samples with primary and secondary sequences.
- 










The script identifies and assesses potential sequence conflicts and contaminants:<br><br>
Cross-Contamination: Maps the distribution of positive control reads across plates and identifies potential cross-contamination sources in the negative controls. This step shows how far on a plate the potential contamination could spread. <br><br>
Conflicting Sequences: Identifies conflicts within a sample where secondary sequences have > 100 reads or 50% or more of the primary sequence read and returns tables listing conflicts at the family and order levels. These tables can be used to recognise samples that may have two large insects plated together (partner’s error) and true symbiont/parasite interactions.
Unique Secondary Sequences: Searches for secondary sequences with more than 50 reads that are not found elsewhere on the plate, indicating a potential true signal. These tables can be used to recognise samples that may have two large insects plated together (partner’s error) and true symbiont/parasite interactions.
Shorter Sequences: Identifies sequences shorter than expected within those without assigned taxonomy and replaces them with the closest matching longer sequence within the sample (on average 100 bp longer; Levenshtein distance < 150). <br><br>
Non-Arthropod Sequences: Replaces all primary non-Arthropod sequences with the Arthropod sequence with the highest read count. Wolbachia, Tardigrades, Rotifers, and Nematodes are retained in the output for further investigation. <br><br>
Quality Scores: Categorises all the retained samples into categories depending on read count and the level of secondary sequence contamination <ins><b>from the same family or order</b></ins>. <br>The main QC report also contains a table showing how many samples were clasified into which category. This information is also save in read_summary_metadata.csv file.

| Score | Category       | No. reads in primary | Secondary sequence assessment [the same family or order]                                | Decision                                |
|-------|----------------|----------------------|--------------------------------------------------------------|-----------------------------------------|
| <b>1</b>     | <i>Perfect</i>        | > 200                | No secondary sequence with 5 or more reads           |YES |
| <b>2</b>     | <i>Almost perfect</i> | 100-200              | No secondary sequence with 5 or more reads           |YES |
| <b>3</b>     | <i>Very good</i>      | < 100                | No secondary sequence with 5 or more reads           |YES |
| <b>4</b>     | <i>Good</i>           | > 200                | At least one secondary sequence with 5 or more reads |NO |
| <b>5</b>     | <i>Ok</i>             | 100-200              | At least one secondary sequence with 5 or more reads |NO |
| <b>6</b>     | <i>Almost ok</i>      | < 100                | At least one secondary sequence with 5 or more reads |NO |
| <b>7</b>     | <i>Need attention</i> | NA                | Conflicts detected in previous steps                     |NO |
| <b>8</b>     | <i>Exclude</i>        | < 15                  | At least one secondary sequence with 3 or reads       |NO |

<br><i>Final assessments and plots </i><br><br>
This part contains tables with percentages of retained samples per partner, partner plate, and UMI plate. <br>
Further, all partner plates and UMI plates are displayed as heatmaps.
