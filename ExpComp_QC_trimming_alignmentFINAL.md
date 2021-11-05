# Express Compare QC, Trimming, Alignment, Downsampling Workflow 

## Author: Lauren Zane; Last updated: 20211105

Bioinformatic tools used to prepare for DESeq2 and GO Analysis

initial quality check: FastQC, MultiQC

quality trimming: Fastp

alignment to reference genome: HiSat2

.sam to .bam conversion/.bam file sorting: SAMTools

note: use $ interactive to access node for jobs; use module avail or module available for list of modules loaded onto 

note: for nearly all jobs use sbatch

#### upload reference genome from 

http://cyanophora.rutgers.edu/Pocillopora_acuta/
http://cyanophora.rutgers.edu/montipora/

###### I stored the reference genome files on my Desktop, changed directories to Desktop, then securely copied the files onto the remote server
```
scp Montipora_capitata_HIv2.assembly.fasta.gz laurenzane@ssh3.hac.uri.edu:/data/putnamlab/shared/Express_Compare/REF
scp Pocillopora_acuta_HIv1.assembly.fasta.gz laurenzane@ssh3.hac.uri.edu:/data/putnamlab/shared/Express_Comapre/REF
```

###### navigate to the RAW directory to access raw files 

```
cd /data/putnamlab/shared/Express_Compare/RAW
```

###### count number of .fast.gz files in directory 
```
ls | wc -l
```
 

###### count raw reads for each .fastq.gz file 

```
zgrep -c "@GWNJ" *.fastq.gz
```

forward raw read filename | number of reads | reverse raw read filename | number of reads
------------------------- | --------------- | ------------------------- | ---------------
1041_R1_001.fastq.gz | 18637582 | 1041_R2_001.fastq.gz | 18637582 
1101_R1_001.fastq.gz | 21636831 | 1101_R2_001.fastq.gz | 21636831
1471_R1_001.fastq.gz | 7350473  | 1471_R2_001.fastq.gz | 7350473
1548_R1_001.fastq.gz | 17651137 | 1548_R2_001.fastq.gz | 17651137
1628_R1_001.fastq.gz | 21647915 | 1628_R2_001.fastq.gz | 21647915
1637_R1_001.fastq.gz | 7188549  | 1637_R2_001.fastq.gz | 7188549
Sample1_R1.fastq.gz | 0 | Sample1_R2.fastq.gz | 0
Sample2_R1.fastq.gz | 0 | Sample2_R2.fastq.gz | 0
Sample3_R1.fastq.gz | 0 | Sample3_R2.fastq.gz | 0
Sample4_R1.fastq.gz | 0 | Sample4_R2.fastq.gz | 0
Sample5_R1.fastq.gz | 0 | Sample5_R2.fastq.gz | 0
Sample6_R1.fastq.gz | 0 | Sample6_R2.fastq.gz | 0

Samples 1-6 did not return the raw read counts

since files are in fastq format, we can count the number of lines and divide by 4 for the number of reads

the less command allows us to view the file, use q to exit 

alternatively, you can count the number of "@" signs to count the number of reads

#### table of counted reads for each sample

```
zgrep -c "@" *.fastq.gz
```

forward raw read filename | number of reads | reverse raw read filename | number of reads
------------------------- | --------------- | ------------------------- | ---------------
1041_R1_001.fastq.gz | 18637582 | 1041_R2_001.fastq.gz | 18637582
1101_R1_001.fastq.gz | 21636831 | 1101_R2_001.fastq.gz | 21636831
1471_R1_001.fastq.gz | 16990028  | 1471_R2_001.fastq.gz | 16990028
1548_R1_001.fastq.gz | 17651137 | 1548_R2_001.fastq.gz | 17651137
1628_R1_001.fastq.gz | 21647915 | 1628_R2_001.fastq.gz | 21647915
1637_R1_001.fastq.gz | 16853810  | 1637_R2_001.fastq.gz | 16853810
Sample1_R1.fastq.gz | 16487768 | Sample1_R2.fastq.gz | 16487768
Sample2_R1.fastq.gz | 10755630 | Sample2_R2.fastq.gz | 10755630
Sample3_R1.fastq.gz | 13534820 | Sample3_R2.fastq.gz | 13534820
Sample4_R1.fastq.gz | 10275396 | Sample4_R2.fastq.gz | 10275396
Sample5_R1.fastq.gz | 16601774 | Sample5_R2.fastq.gz | 16601774
Sample6_R1.fastq.gz | 13440121 | Sample6_R2.fastq.gz | 13440121


### quality control and trimming in RAW directory
##### 1. initial quality check on raw reads
##### 2. trimming for quality
##### 3. second quality check after trimming

#### run fastqc on raw reads in RAW directory

```
$ interactive
module load FastQC/0.11.8-Java-11
fastqc ./*.fastq.gz
```

#### run multiqc on fastqc files in RAW directory

```
$ interactive
module load MultiQC/1.9-intel-2020a-Python-3.8.2
multiqc ./ 
```

#### rename multiqc directory and report using the move command in output directory
```
mv multiqc_data multiqc_data_raw  
mv multiqc_report.html multiqc_report_raw.html
```
#### create directory within Express_Compare called "data"
```
mkdir data
```
#### create subdirectory within "data" called "cleaned_reads"
```
cd /data/putnamlab/shared/Express_Compare/data
mkdir cleaned_reads 
```
#### rename Sample 1-6 R1/R2 files as  Sample1-6 R1/R2_001.fastq.gz files using mv command in RAW directory
```
mv Sample1_R1.fastq.gz Sample1_R1_001.fastq.gz 
```
#### run fastp for practice using 1041, 1101, Sample1 and Sample2 with only the phred score argument in RAW directory
```
sh -c 'for file in "1041" "1101" "Sample1" "Sample2"
> do
> fastp --in1 ${file}_R1_001.fastq.gz --in2 ${file}_R2_001.fastq.gz --out1 ${file}_R1_001.clean.fastq.gz --out2 ${file}_R2_001.clean.fastq.gz --qualified_quality_phred 20 
> done'
```
#### run fastp using all other arguments in RAW directory

```
$ interactive 
module load fastp/0.19.7-foss-2018b
sh -c 'for file in "1041" "1101" "1471" "1548" "1628" "1637" "Sample1" "Sample2" "Sample3" "Sample4" "Sample5" "Sample6"
do 
fastp --in1 ${file}_R1_001.fastq.gz 
--in2 ${file}_R2_001.fastq.gz 
--out1 ${file}_R1_001.clean.fastq.gz 
--out2 ${file}_R2_001.clean.fastq.gz 
#--failed_out ${file}_failed.txt 
--qualified_quality_phred 20 
--unqualified_percent_limit 10 
--length_required 100 
--detect_adapter_for_pe 
--cut_right 
--cut_right_window_size 5 
--cut_right_mean_quality 20
done'
```

###### note that the argument --failed_out ${file}_failed.txt did not work, so fastp was run without it

#### move .clean.fastq.gz files to /data/putnamlab/shared/Express_Compare/QC/trimmed
```
zgrep -c "@" *.clean.fastq.gz
```
#### create another table for counted reads in "trimmed" directory

forward raw read filename | number of reads | reverse raw read filename | number of reads
------------------------- | --------------- | ------------------------- | ---------------
1041_R1_001.clean.fastq.gz | 13848286 | 1041_R2_001.clean.fastq.gz | 13848286
1101_R1_001.clean.fastq.gz | 17311077 | 1101_R2_001.clean.fastq.gz | 17311077
1471_R1_001.clean.fastq.gz | 11754513 | 1471_R2_001.clean.fastq.gz | 11754513
1548_R1_001.clean.fastq.gz | 13872075 | 1548_R2_001.clean.fastq.gz | 13872075
1628_R1_001.clean.fastq.gz | 16980267 | 1628_R2_001.clean.fastq.gz | 16980267
1637_R1_001.clean.fastq.gz | 12197956 | 1637_R2_001.clean.fastq.gz | 12197956
Sample1_R1_001.clean.fastq.gz | 12023326 | Sample1_R2_001.clean.fastq.gz | 12023326
Sample2_R1_001.clean.fastq.gz | 7765823 | Sample2_R2_001.clean.fastq.gz | 7765823
Sample3_R1_001.clean.fastq.gz | 9474639 | Sample3_R2_001.clean.fastq.gz | 9474639
Sample4_R1_001.clean.fastq.gz | 7236465 | Sample4_R2_001.clean.fastq.gz | 7236465
Sample5_R1_001.clean.fastq.gz | 12227119 | Sample5_R2_001.clean.fastq.gz | 12227119
Sample6_R1_001.clean.fastq.gz | 9922140 | Sample6_R2_001.clean.fastq.gz | 9922140



## use fastqc/multiqc to assess quality in "trimmed" directory
```
$ interactive
module load FastQC/0.11.8-Java-11
fastqc ./*.fastq.gz
module load MultiQC/1.9-intel-2020a-Python-3.8.2
multiqc ./
```

#### indexing using Hisat2--build in the "REF" directory
```
$ interactive
module load HISAT2/2.2.1-foss-2019b
```

###### unzip .fastq.gz files in "REF" directory, HiSat2--build can only use fasta files 

```
gunzip Montipora_capitata_HIv2.assembly.fastq.gz
gunzip Pocillopora_acuta_HIv1.assembly.fastq.gz
```

###### use HiSat2-build to build index for Mcap and PAcuta genomes
```
hisat2-build -f REF/Montipora_capitata_HIv2.assembly.fasta REF/MCap
hisat2-build -f REF/Pocillopora_acuta_HIv1.assembly.fasta REF/MCap
```
#### staying in the "REF" directory, symbolically link fastq files  
```
ln -s /data/putnamlab/shared/Express_Compare/data/cleaned_reads/*fastq* ./
```

#### HiSat2 Alignment for MCap
```
#!/bin/bash
#SBATCH --job-name="ZaneMCapAlignment2" 
#SBATCH -t 100:00:00 
#SBATCH --export=NONE 
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=laurenzane@uri.edu
#SBATCH -D /data/putnamlab/shared/Express_Compare/REF 
#SBATCH --exclusive #SBATCH -c 20
module load SAMtools/1.10-GCC-8.3.0
module load HISAT2/2.2.1-foss-2019b
ids=("1101" "1628" "1548" "Sample4" "Sample5" "Sample6")
for file in "${ids[@]}"; 
do hisat2 -p 20 --rf --dta -q -x MCap/MCap -1 "$file"_R1_001.clean.fastq.gz -2 "$file"_R2_001.clean.fastq.gz -S "$file".sam; 
samtools sort -@ 20 -o "$file".bam "$file".sam; rm "$file".sam; done
echo "HISAT2 complete for" "$file" "$(date)"
```

#### HiSat2 Alignment for PAcuta
```
#!/bin/bash
#SBATCH --job-name="ZanePAcutaAlignment2"
#SBATCH -t 100:00:00
#SBATCH --export=NONE
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=laurenzane@uri.edu
#SBATCH -D /data/putnamlab/shared/Express_Compare/REF
#SBATCH --exclusive #SBATCH -c 20
module load SAMtools/1.10-GCC-8.3.0
module load HISAT2/2.2.1-foss-2019b
ids=("1041" "1471" "1637" "Sample1" "Sample2" "Sample3")
for file in "${ids[@]}"; 
do hisat2 -p 20 --rf --dta -q -x PAcuta/Pacuta -1 "$file"_R1_001.clean.fastq.gz -2 "$file"_R2_001.clean.fastq.gz -S "$file".sam; 
samtools sort -@ 20 -o "$file".bam "$file".sam; rm "$file".sam; done
echo "HISAT2 complete for" "$file" "$(date)"
```
#### count reads in .bam files 
```
samtools view -c $file.bam
```
Species | Identifier | original number of counted reads | p value for downsampling | downsampled number of reads
------- | ---------- | ----------------------- | ------------------------ | -----------------------
P. Acuta | Sample1 | 33880045 | 0.56 | 18980246
P. Acuta | Sample2 | 19067556 | 1 | 19067556
P. Acuta | Sample3 | 23507575 | 0.81 | 19040496
M. Capitata | Sample 4 | 21361627 | 0.89 | 19005470
M. Capitata | Sample 5 | 36480300 | 0.52 | 18975898
M. Capitata | Sample 6 | 29665414 | 0.64 | 19004386
P. Acuta | 1041 | 30122944 | 0.63 | 18975322
P. Acuta | 1471 | 25355395 | 0.75 | 19017022
P. Acuta | 1637 | 26253673 | 0.72 | 18894850
M. Capitata | 1101 | 39764959 | 0.47 | 18684042
M. Capitata | 1628 | 38182073 | 0.5 | 19087149
M. Capitata | 1548 | 30727605 | 0.62 | 19045874

#### GATK Downsampling to smallest number of reads, which is P. Acuta Sample 2, 19067556
```
java -jar $EBROOTPICARD/picard.jar DownsampleSam INPUT= "$file".bam O="$file".downsampled.bam P=0.56
```
#### again, count reads to make sure that downsampling worked
 ```
 samtools view -c "$file".downsampled.bam
 ```
 
#### you now have downsampled .bam files ready for Stringtie! 

currently these .bam files reside in /data/putnamlab/shared/Express_Compare/data/hisat2_alignment


