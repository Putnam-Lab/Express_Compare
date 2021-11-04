# M. capitata, P. acuta Poly A versus RiboFree QC, Trimming, Alignment Workflow 

#### Author: Lauren Zane; Last updated: 20211031

#### Bioinformatic tools used to prepare for DESeq2 and GO Analysis
#### initial quality check: FastQC, MultiQC
#### quality trimming: Fastp
#### alignment to reference genome: HiSat2
#### note: use $ interactive to access node for jobs; use module avail or module available for list of modules loaded onto 

#### upload reference genome from 
http://cyanophora.rutgers.edu/Pocillopora_acuta/
http://cyanophora.rutgers.edu/montipora/

###### I stored the reference genome files on my Desktop, changed directories to Desktop, then securely copied the files onto the remote server
scp Montipora_capitata_HIv2.assembly.fasta.gz laurenzane@ssh3.hac.uri.edu:/data/putnamlab/shared/Express_Compare/REF
scp Pocillopora_acuta_HIv1.assembly.fasta.gz laurenzane@ssh3.hac.uri.edu:/data/putnamlab/shared/Express_Comapre/REF

###### navigate to the RAW directory to access raw files 

cd /data/putnamlab/shared/Express_Compare/RAW

###### count number of .fast.gz files in directory 

ls | wc -l 

###### 24 files 

### upload raw sequence data and reference genome to local server

###### not sure where data resides 

###### count raw reads for each .fastq.gz file 

zgrep -c "@GWNJ" *.fastq.gz

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

###### it is problematic that Samples 1-6 do not return the raw read counts
###### since files are in fastq format, we can count the number of lines and divide by 4 for the number of reads
###### the less command allows us to view the file, use q to exit 
less Sample1_R1.fastq.gz

###### table 2 using "@" to count the number of reads

zgrep -c "@" *.fastq.gz

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

#### create a conda environment

conda create -n Express_Compare

environment location: /home/laurenzane/miniconda3/envs/Express_Compare

conda activate Express_Compare

#### install QC/trimming modules in the conda environment

conda install -c bioconda fastqc
conda install -c bioconda multiqc
conda install -c bioconda fastp

#### quality control and trimming in RAW directory
##### 1. initial quality check on raw reads
##### 2. trimming for quality
##### 3. second quality check after trimming

#### run fastqc on raw reads in RAW directory

$ interactive
module load FastQC/0.11.8-Java-11
fastqc ./*.fastq.gz

#### run multiqc on fastqc files in RAW directory

$ interactive
module load MultiQC/1.9-intel-2020a-Python-3.8.2
multiqc ./ 

#### rename multiqc directory and report using the move command in output directory

mv multiqc_data multiqc_data_raw  
mv multiqc_report.html multiqc_report_raw.html

#### create directory within Express_Compare called "data"

mkdir data

#### create subdirectory within "data" called "cleaned_reads"

cd /data/putnamlab/shared/Express_Compare/data
mkdir cleaned_reads 

#### rename Sample 1-6 R1/R2 files as  Sample1-6 R1/R2_001.fastq.gz files using mv command in RAW directory

mv Sample1_R1.fastq.gz Sample1_R1_001.fastq.gz 

#### run fastp for practice using 1041, 1101, Sample1 and Sample2 with only the phred score argument in RAW directory

sh -c 'for file in "1041" "1101" "Sample1" "Sample2"
> do
> fastp --in1 ${file}_R1_001.fastq.gz --in2 ${file}_R2_001.fastq.gz --out1 ${file}_R1_001.clean.fastq.gz --out2 ${file}_R2_001.clean.fastq.gz --qualified_quality_phred 20 
> done'

#### run fastp using all other arguments in RAW directory


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

###### note that the argument --failed_out ${file}_failed.txt did not work, so fastp was run without it

#### move .clean.fastq.gz files to /data/putnamlab/shared/Express_Compare/QC/trimmed

zgrep -c "@" *.clean.fastq.gz

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
#### create the subdirectory "alignment" in the "data" directory 





#### indexing using Hisat2--build in the REF directory

interactive
module load HISAT2/2.2.1-foss-2019b

###### unzip .fastq.gz files, HiSat2--build can only use fasta files 


#### navigate to the hisat2_alignment subdirectory, symbolically link fastq files  
ln -s /data/putnamlab/shared/Express_Compare/data/cleaned_reads/*fastq* ./

#!/bin/bash
nano ZaneMCapAlignment.sh
#SBATCH --job-name="ZaneMCapAlignment" 
#SBATCH -t 100:00:00 
#SBATCH --export=NONE 
#SBATCH --mail-type=BEGIN,END,FAIL 
#SBATCH -D /data/putnamlab/shared/Express_Compare/REF 
#SBATCH --exclusive #SBATCH -c 20
module load SAMtools/1.10-GCC-8.3.0
module load HISAT2/2.2.1-foss-2019b
ids=("1101" "1628" "1548" "Sample4" "Sample5" "Sample6")
for file in "${ids[@]}"; do hisat2 -p 20 --rf --dta -q -x MCap/MCap -1 "$file"_R1_001.clean.fastq.gz -2 "$file"_R2_001.clean.fastq.gz -S "$file".sam; samtools sort -@ 20 -o bam/paired/"$file".bam "$file".sam; rm "$file".sam; done
echo "HISAT2 complete for" "$file" "$(date)"


####
#!/bin/bash
#SBATCH --job-name="ZanePAcutaAlignment"
#SBATCH -t 100:00:00
#SBATCH --export=NONE
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=laurenzane@uri.edu
#SBATCH -D /data/putnamlab/shared/Express_Compare/REF
#SBATCH --exclusive #SBATCH -c 20
module load SAMtools/1.10-GCC-8.3.0
module load HISAT2/2.2.1-foss-2019b
ids=("1041" "1471" "1637" "Sample1" "Sample2" "Sample3")
for file in "${ids[@]}"; do hisat2 -p 20 --rf --dta -q -x PAcuta/Pacuta -1 "$file"_R1_001.clean.fastq.gz -2 "$file"_R2_001.clean.fastq.gz -S "$file".sam; samtools sort -@ 20 -o bam/paired/"$file".bam "$file".sam; rm "$file".sam; done
echo "HISAT2 complete for" "$file" "$(date)"
















