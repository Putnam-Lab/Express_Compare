# Express Compare Assembly 
## Author: Lauren Zane; Last Updated: 2021/11/07
### Bioinformatic tools used in analysis: 
Transcript assemble and quantification: StringTie

##### import GFF files from http://cyanophora.rutgers.edu/montipora/ and http://cyanophora.rutgers.edu/Pocillopora_acuta/

download GFF files from Rutgers website onto local server; securely copy onto Putnam Lab remote server. Verify data quality using mdsum5.
To use mdsum5, first store md5checksum in a file, then verify quality of files. You should see "OK" if the files copied properly.
Unzip .gz files because Stringtie will only run with a .gff file.

```
scp Montipora_capitata_HIv2.genes.gff3.gz laurenzane@ssh3.hac.uri.edu:/data/putnamlab/shared/Express_Compare/REF
scp Pocillopora_acuta_HIv1.genes.gff3.gz laurenzane@ssh3.hac.uri.edu:/data/putnamlab/shared/Express_Compare/REF
md5sum *.gff3.gz > raw_checksum.md5
md5sum -c raw_checksum.md5
gunzip Montipora_capitata_HIv2.genes.gff3.gz
gunzip Pocillopora_acuta_HIv1.genes.gff3.gz

```

##### prepare for stringtie by making a directory for gene abumdance files 

create stringtie_assembly directory in Express_Compare directory. Create symbolic links 
for the reference genomes and the .bam files produced from the HiSat2 alignment step.
The refence genomes and bam files will be symbolically linked to the stringtie_assembly directory.
note: Sample2.bam, which was the number of reads I downsampled to was renamed Sample2.downsampled.bam to match the other bam files.
note2: I also created a separate subdirectory for downsampled bam files within the /Express_Compare/data directory called "downsampled"

```
cd /data/putnamlab/shared/Express_Compare
mkdir stringtie_assembly
cd stringtie_assembly
ln -s /data/putnamlab/shared/Express_Compare/REF/Montipora_capitata_HIv2.genes.gff3 ./
ln -s /data/putnamlab/shared/Express_Compare/REF/Pocillopora_acuta_HIv1.genes.gff3 ./
ln -s /data/putnamlab/shared/Express_Compare/data/downsampled/*.bam ./

```
#### reference guided assembly for gene abundance 

create subdirectory "gene_abundance" within the directory stringtie_assembly. 

```
mkdir gene_abundance
cd ../
```

test code for stringtie 

```
stringtie -A gene_abundance$1101.gene_abundance.tab --rf -e -G Montipora_capitata_HIv2.genes.gff3 -o ${i}.gtf 1101.downsampled.bam 
```

test code for for loop with stringtie

```
ids=(1101" "Sample4")
for file in "${ids[@]}"
do
stringtie -A gene_abundance/"$file".gene_abundance.tab --rf -e -G Montipora_capitata_HIv2.genes.gff3 -o "$file".gtf "$file".downsampled.bam
done
```

creation of stringtie assembly script for MCap

```
nano ZaneStringtieAssemblyMCap.sh

#!/bin/bash
#SBATCH --job-name="ZaneStringtieAssemblyMCap" 
#SBATCH -t 100:00:00 
#SBATCH --export=NONE 
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=laurenzane@uri.edu
#SBATCH -D /data/putnamlab/shared/Express_Compare/stringtie_assembly 
#SBATCH --exclusive #SBATCH -c 20

module load StringTie/2.1.3-GCC-8.3.0

ids=("1101" "1628" "1548" "Sample4" "Sample5" "Sample6")
for file in "${ids[@]}"
do
stringtie -A gene_abundance/"$file".gene_abundance.tab --rf -e -G Montipora_capitata_HIv2.genes.gff3 -o "$file".gtf "$file".downsampled.bam
done
```

creation of stringtie assembly script for PAcuta

```
nano ZaneStringtieAssemblyPAcuta.sh

#!/bin/bash
#SBATCH --job-name="ZaneStringtieAssemblyPAcuta" 
#SBATCH -t 100:00:00 
#SBATCH --export=NONE 
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=laurenzane@uri.edu
#SBATCH -D /data/putnamlab/shared/Express_Compare/stringtie_assembly 
#SBATCH --exclusive #SBATCH -c 20

module load StringTie/2.1.3-GCC-8.3.0

ids=("1041 "1471" "1637" "Sample1" "Sample2" "Sample3")
for file in "${ids[@]}"
do
stringtie -A gene_abundance/"$file".gene_abundance.tab --rf -e -G Pocillopora_acuta_HIv1.genes.gff3 -o "$file".gtf "$file".downsampled.bam
done
```
#### use GFF compare to compare the merged GTF to our reference genome in the "stringtie_assembly" directory
```
module load GffCompare/0.12.1-GCCcore-8.3.0

gffcompare -r Montipora_capitata_HIv2.genes.gff3 -G -o MCap_merged MCap_stringtie_merged.gtf
gffcompare -r Pocillopora_acuta_HIv1.genes.gff3 -G -o PAcuta_merged PACuta_stringtie_merged.gtf


```
stringtie --merge -p 8 -G Pocillopora_acuta_HIv1.genes.gff3 -o PACuta_stringtie_merged.gtf PAcuta_merge.txt

stringtie --merge -p 8 -G Montipora_capitata_HIv2.genes.gff3 -o MCap_stringtie_merged.gtf MCap_merge.txt
```

#### compiling GTF files into gene and transcript matrices, compatible with DESeq2

during the final step of the Stringtie workflow, you will first need to create a .txt file using that contains the sample names
and the full path to the GTF files

example: 

1101 /data/putnamlab/shared/Express_Compare/stringtie_assembly/1101.gtf

1628 /data/putnamlab/shared/Express_Compare/stringtie_assembly/1628.gtf

1548 /data/putnamlab/shared/Express_Compare/stringtie_assembly/1548.gtf

Sample4 /data/putnamlab/shared/Express_Compare/stringtie_assembly/Sample4.gtf

Sample5 /data/putnamlab/shared/Express_Compare/stringtie_assembly/Sample5.gtf

Sample6 /data/putnamlab/shared/Express_Compare/stringtie_assembly/Sample6.gtf



```
nano MCap_prepDE.txt
prepDE.py -g MCap_gene_count_matrix.csv -i ./MCap_prepDE.txt

nano PAcuta_prepDE.txt
prepDE.py -g PAcuta_gene_count_matrix.csv -i ./PAcuta_prepDE.txt
```







