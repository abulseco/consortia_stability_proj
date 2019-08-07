# Consortia Stability Project (Metcalf 2019)

#### Updated 08/06/2019 - Ashley Bulseco

This code is specific to an undergraduate Metcalf research project that aims to investigate the stability of a microbial consortia isolated from salt marsh sediments over time. This work is in collaboration with [J. Bowen](https://jb2032.wixsite.com/bowenlab) and [J. Vineis](https://github.com/jvineis) of Northeastern University. This workflow uses Illumina-utils and resolves nodes using minimum entropy decomposition from [Eren et al. 2015](https://www.nature.com/articles/ismej2014195). 

## 1. Requirements for this workflow
* Illumina-utils found [here](https://github.com/merenlab/illumina-utils) 
* Oligotyping pipeline found [here](http://merenlab.org/2014/08/16/installing-the-oligotyping-pipeline/)
* Levenshtein module found [here](https://pypi.org/project/python-Levenshtein/)
* Download data and associated mapping file (available in this repo) 

To download Illumina-utils: 
```
pip install illumina-utils
```
To download oligotyping pipeline:
```
sudo pip install oligotyping
```
To download Levenshtein module
```
pip install python-levenshtein
```

## 2. Make a working directory
In this case, the working directory is within the documents folder, which can be changed according to your needs. You should put your mapping file (barcode_to_sample.txt) and sequence data (RAW_data folder) located in this repo, and navigate to that folder.  
```
mkdir ~/Documents/consortia_med
cd ~/Documents/consortia_med
ls # to make sure your data and mapping file are located in the correct folder
mkdir demultiplex # to put demultiplexed data in
```

## 3. Demultiplex your data
*Note: This may have already been done for you prior to receiving the data, but you can practice this step if you would like*

The demultiplexing step allows you to reference what sequences came from which samples using unique barcodes ligated to each individual sample. This is important, because all of your samples had to be sequenced together. The "barcode_to_sample.txt" mapping file provides this information. After demultiplexing, you can then generate a report that provides information on which paired reads go together. 
```
iu-demultiplex -s barcode_to_sample.txt --r1 Undetermined_S0_L001_R1_001.fastq --r2 Undetermined_S0_L001_R2_001.fastq --index Undetermined_S0_L001_I1_001.fastq -o demultiplexed -x 

iu-gen-configs 00_DEMULTIPLEXING_REPORT
```

## 4. Merging partially overlapping paired reads
The iu-merge-pairs function will merge overlapping reads and the enforce-Q30-check will trim low quality scores on the non-overlapping part of the reads. Note that if your fragments are larger and the paired end reads do not overlap, then the reads will stay independent and you will have to do your own quality trimming.

The merging is only working for overlapping reads, which is what we are doing today. However if the merging doesn't happen because the pairs don't overlap, then you would just use quality filtering and visualize with iu-filter-quality-minoche, there is also iu-filter-quality-bokulich from the illumina-utils. We aren't doing this today because we will be merging. enforce-Q30-check is looking at quality on the ends, not just where the reads are overlapping. 
```
ls *.ini | sed 's/\.ini//g' > samples.txt  # "ls" lists all file that end with an .ini, "sed" is a find and replace tool, and the ">" creates a new file. We are finding "*\.ini" and replacing it with "", use forward slash before because unix hates dots

gunzip *.ini # we have to unzip the fastq files to run the following code

screen # opening a screen will allow merging to happen in the background even if your computer falls asleep

for i in `cat samples.txt`; do iu-merge-pairs $i'.ini' -o $i'-merged' --enforce-Q30-check; done
```
Then you can use "more" or "head" to look at the files containing the produced stats. The following outlines some of the potential read outcomes:
* FAILED are reads that failed to merge
* FAILED_Q30 are reads that failed quality check
* FAILED_WITH_Ns whether you get rid of reads with Ns
* MERGED success! If you inspect this file, you will see that it specifies the number of mismatches for a single read. If there is a mismatch, it takes the read for which the quality score is highest. The stats file gives more information.
* MISMATCHES_BREAKDOWN mismatches found in merged region between reads

## 5. Filter out merged reads that have more than three mismatches
This will result in a directory with data that have been merged and quality filtered. The next step is to use MED.
```
for i in `cat samples.txt`; do iu-filter-merged-reads $i'-merged_MERGED' -m 3; done
```



