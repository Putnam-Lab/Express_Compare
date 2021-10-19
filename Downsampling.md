
## navigated to directory where example bam files are located
[laurenzane@ssh3 ~]$ cd /data/putnamlab/shared/example_bam


## listed files within example_bam directory

[laurenzane@ssh3 example_bam]$ ls
1041.bam  1471.bam  1628.bam  bam.read.counts  slurm-92786.out
1101.bam  1548.bam  1637.bam  downsample.sh    slurm-92834.out

## loaded modules. Note: these are different than the ones from the GATK_downsampling example!
if you try to load the modules used in the example, there is module incompatibility
the command "module available" in bash allows you to see which modules have been loaded onto Andromeda

 Big note! working in the environmental variable is the major change from the downsampling example found in PutnamLab/Lab_Management
 if you do not work in the environmental variable, $EBROOTPICARD, it will let you know that picard.jar is "not found"

[laurenzane@ssh3 example_bam]$ module load GATK/4.2.0.0-GCCcore-10.2.0-Java-11
WARNING: GATK v4.2.0.0 support for Java 11 is in beta state. Use at your own risk.

[laurenzane@ssh3 example_bam]$ module load SAMtools/1.12-GCC-10.2.0
[laurenzane@ssh3 example_bam]$ module load picard/2.25.1-Java-11
To execute picard run: java -jar $EBROOTPICARD/picard.jar

use the echo function to print a string into the terminal 

[laurenzane@ssh3 example_bam]$ echo "original bam read number" 
original bam read number

## count original number of reads

[laurenzane@ssh3 example_bam]$ samtools view -c 1041.bam
31204984

## downsampling
this code works! thanks Hollie. Note that it will tell you that Picard's command line is changing
 to DownsampleSam -INPUT 1041.bam -O 1041.downsampled.bam -P 0.2 
as of 20211019, this code does not work! 
P = 0.2 means that 20% of reads will be kept aka "downsampled to 0.2"


[laurenzane@ssh3 example_bam]$ java -jar $EBROOTPICARD/picard.jar DownsampleSam INPUT=1041.bam O=1041.downsampled.bam P=0.2

## count reads in downsampled files
[laurenzane@ssh3 example_bam]$ echo "downsampled bam read number" 
downsampled bam read number
[laurenzane@ssh3 example_bam]$ samtools view -c 1041.downsampled.bam
6238951

## for loop counts all files ending in .bam 
echo was useful because without echo I wouldn't know
which counts belonged to each file

[laurenzane@ssh3 example_bam]$ for file in *.bam 
> do
> echo $file
> samtools view -c $file
> done

## number of reads in each .bam file
bam file name | number of reads 
------------- | ---------------
1041.bam | 31204984
1041.downsampled.bam | 6238951
1101.bam | 46211600
1471.bam | 26845644
1548.bam | 36612211
1628.bam | 45841870
1637.bam | 27572459

 remove 1041.downsampled.bam so I don't accidentally downsample it
[laurenzane@ssh3 example_bam]$ rm 1041.downsampled.bam

 downsampling performed individually for all samples; each run takes about two minutes
 at the moment, I am unable to use a for loop for downsampling because the program can only handle one bam file at a time
 additionally, different P values are used to attain the same number of reads, so I am unable to do this in a loop 

 1041.bam
[laurenzane@ssh3 example_bam]$ java -jar $EBROOTPICARD/picard.jar DownsampleSam INPUT=1041.bam O=1041.downsampled.bam P=0.2

 1101.bam
[laurenzane@ssh3 example_bam]$ java -jar $EBROOTPICARD/picard.jar DownsampleSam INPUT=1101.bam O=1101.downsampled.bam P=0.13

 1471.bam 
[laurenzane@ssh3 example_bam]$ java -jar $EBROOTPICARD/picard.jar DownsampleSam INPUT=1471.bam O=1471.downsampled.bam P=0.22

 1548.bam
[laurenzane@ssh3 example_bam]$ java -jar $EBROOTPICARD/picard.jar DownsampleSam INPUT=1548.bam O=1548.downsampled.bam P=0.16

 1628.bam
[laurenzane@ssh3 example_bam]$ java -jar $EBROOTPICARD/picard.jar DownsampleSam INPUT=1628.bam O=1628.downsampled.bam P=0.13

 1637.bam 
[laurenzane@ssh3 example_bam]$ java -jar $EBROOTPICARD/picard.jar DownsampleSam INPUT=1637.bam O=1637.downsampled.bam P=0.22

## count .downsampled.bam files using for loop

 test

for file in *.downsampled.bam
do
echo $file
done

for file in *.downsampled.bam
do
echo $file
samtools view -c $file
done

## record reads in each .downsampled.bam file
downsampled bam name | number of reads
---------------------| ---------------
1041.downsampled.bam | 6238951
1101.downsampled.bam | 6007810
1471.downsampled.bam | 5904109
1548.downsampled.bam | 5852572
1628.downsampled.bam | 5954221
1637.downsampled.bam | 6064188





