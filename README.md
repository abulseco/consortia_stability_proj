# Consortia Stability Project (Metcalf 2019)

#### Updated 08/06/2019 - Ashley Bulseco

This code is specific to an undergraduate Metcalf research project that aims to investigate the stability of a microbial consortia isolated from salt marsh sediments over time. This work is in collaboration with [J. Bowen](https://jb2032.wixsite.com/bowenlab) and [J. Vineis](https://github.com/jvineis) of Northeastern University. This workflow uses Illumina-utils and resolves nodes using minimum entropy decomposition from [Eren et al. 2015](https://www.nature.com/articles/ismej2014195). 

## 1. Requirements for this workflow
* Illumina-utils found [here](https://github.com/merenlab/illumina-utils) 
* Oligotyping pipeline found [here](http://merenlab.org/2014/08/16/installing-the-oligotyping-pipeline/)
* Download data and associated mapping file (available in this repo) 

To download Illumina-utils: 
```
pip install illumina-utils
```
To download oligotyping pipeline:
```
sudo pip install oligotyping
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

