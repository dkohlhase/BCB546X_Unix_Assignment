# BCB 546X - Unix Assignment
### Due: September 21 by 17:00 
### Daniel Kohlhase

[Data Inspection](#data-inspection)

[Data Processing](#data-processing)

<br>

## Data Inspection

### File Size

	$ ls -lh fang_et_al_genotypes.txt snp_position.txt

> -rw-r—r—. 1 kohlhase domain users 11M Aug 31 13:33 `fang_et_al_genotypes.txt`

> -rw-r—r—. 1 kohlhase domain users 81K Aug 31 13:33 `snp_position.txt`

The genotype file is 11 Mb and the snp file is 81 kb

<br>

### Lines, Words, Characters

	$ wc fang_et_al_genotypes.txt snp_position.txt

|Lines		|Words		|Characters	|File	
| --- 		| --- 		| --- 			| ---
|2783		|2744038	|11051939		|`fang_et_al_genotypes.txt`
|984		|13198		|82763			|`snp_position.txt`
|3767		|2757236	|11134702		|`total`

The genotype file has 2783 lines and the snp file has 984 lines.

<br>

### Columns

	$ awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt
>	986

The genotype file has 986 columns.

	$ awk -F "\t" '{print NF; exit}' snp_position.txt
>	15

The snp file has 15 columns.

<br>
<br>
<br>

## Data Processing

### Extract maize data from genotype file 

First we need the column headers.

	$ head -n 1 fang_et_al_genotypes.txt > maize.txt

We can extract maize data in one fell swoop.
	
	$ grep -e "ZMMIL" -e "ZMMLR" -e "ZMMMR" fang_et_al_genotypes.txt >> maize.txt
	$ wc maize.txt

|Lines		|Words		|Characters	|File	
| --- 		| --- 		| --- 			| ---
|1574		|1551964	|6250961		|`maize.txt`

Double check the unique entries in column 3 of maize.txt while counting the number of each entry

	$ cut -f 3 maize.txt | sort | uniq -c

|1		|Group	
| --- 	| ---	
|290	|ZMMIL
|1256	|ZMMLR
|27		|ZMMMR

<br>

### Extract teosinte data from genotype file

	$ head -n 1 fang_et_al_genotypes.txt > teosinte.txt
	$ grep  -e "ZMPBA" -e "ZMPIL" -e "ZMPJA" fang_et_al_genotypes.txt >> teosinte.txt
	$ wc teosinte.txt 
	
|Lines		|Words		|Characters	|File	
| --- 		| --- 		| --- 			| ---
|976		|962336	|3884185		|`teosinte.txt`

Double check the unique entries in column 3 of teosinte.txt while counting the number of each entry.

	$ cut -f 3 teosinte.txt | sort | uniq -c
|1		|Group
| --- 	| ---	
|900	|ZMPBA
|41		|ZMPIL
|34		|ZMPJA

<br>

### Transpose the maize and teosinte files

	$ awk -f transpose.awk maize.txt > maize_transposed.txt
	$ wc maize_transposed.txt
	
|Lines		|Words		|Characters	|File	
| --- 		| --- 		| --- 			| ---
|986		|1551964	|6250961		|`maize_transposed.txt`

	$ awk -f transpose.awk teosinte.txt > teosinte_transposed.txt
	$ wc teosinte_transposed.txt

|Lines		|Words		|Characters	|File	
| --- 		| --- 		| --- 			| ---
|986		|962336	|3884185		|`teosinte_transposed.txt`

The number of lines has changed in both files and now match. The number of words and characters remained the same for the respective files, which is what we would expect because all we did was rearrange (transpose) the file.

<br>

### Create header and information files from the genotype file

The number of lines is different for the genotype and snp files (986 vs 984). The genotype files have an additional two lines containing additional sample information. 

Create two new files. 

The first file is the transposed genotype files **minus the top two lines** (*_short.txt). 

	$ tail -n +4 maize_transposed.txt > maize_tran_short.txt

The second file is a header file which is **just the top two lines** of the transposed genotype files (*_joined.txt), .

	$ head -n 3 maize_transposed.txt > maize_header.txt

Manually add labels for the SNP_ID, Chromosome, Position in the first three columns of the header file.
	
	$ vi maize_header.txt
	
<br>

### Extract and rearange SNP information

	$ cut -f 1 snp_position.txt > short_snp.txt
	$ cut -f 3 snp_position.txt > short_snp_chrom.txt
	$ cut -f 4 snp_position.txt > short_snp_pos.txt
	
The files are pasted together in the desired format of the final file (SNP_ID, Chromosome, Position).
	
	$ paste short_snp.txt short_snp_chrom.txt > short_snp_plus_chrom.txt
	$ paste short_snp_plus_chrom.txt short_snp_pos.txt  > short_snp_all.txt

Before joining the SNP information with the genotype information, both files need to be sorted so the SNP_IDs are aligned.

	$ sort -k1,1 short_snp_all.txt > short_snp_all_sort.txt 
	$ sort -k1,1 maize_tran_short.txt > maize_tran_short_sort.txt
	
<br>

### Combine the SNP and genotype information

	$ join -1 1 -2 1 -t $'\t' -e 'empty' short_snp_all_sort.txt maize_tran_short.txt > maize_join_full.txt	
	
Append to the header file.
	
	$ cat maize_header.txt maize_join_full.txt > MAIZE.txt

<br>

### Extract data for input files

10 files (1 for each chromosome) with SNPs ordered based on increasing position values and with missing data encoded by this symbol: ?

	$ sort -k2,2n -k3,3n MAIZE.txt > MAIZE_sort.txt 
	$ awk '{ if ($2 == 1 && $3 !~ /multiple/) { print } }' MAIZE_sort.txt > maize_chrom_01_IN_headless.txt
	$ cat maize_header.txt maize_chrom_01_IN_headless.txt > maize_chrom_01_increase.txt

The 'awk' command followed by appending the subset to the maize header file is **repeated for each chromosome**.

<br>	

10 files (1 for each chromosome) with SNPs ordered based on decreasing position values and with
missing data encoded by this symbol: -

	$ sort -k2,2n -k3,3nr MAIZE.txt > MAIZE_reverse.txt
	$ sed 's/?/-/g' MAIZE_reverse.txt > MAIZE_reverse_sort.txt
	$ awk '{ if ($2 == 1 && $3 !~ /multiple/) { print } }' MAIZE_reverse_sort.txt > maize_chrom_01_RV_headless.txt
	$ cat maize_header.txt maize_chrom_01_RV_headless.txt > maize_chrom_01_reverse.txt

The addition of the 'sed' program substitutes -/- for ?/? to identify missing information.

<br>

1 file with all SNPs with unknown positions in the genome (these need not be ordered in any particular way)

	$ awk '{ if ($2 ~ /unknown/ || $3 ~ /unknown/) { print } }' MAIZE_sort.txt > maize_chrom_unk_headless.txt
	$ cat maize_header.txt maize_chrom_unk_headless.txt > maize_chrom_unk.txt
	
1 file with all SNPs with multiple positions in the genome (these need not be ordered in any particular way)
	
	$ awk '{ if ($2 ~ /multiple/ || $3 ~ /multiple/) { print } }' MAIZE_sort.txt > maize_chrom_mult_headless.txt
	$ cat maize_header.txt maize_chrom_mult_headless.txt > maize_chrom_mult.txt

The same 'awk' command is used to extract the unknown and multiple SNP information except adapted to search for an alphanumeric string rather than a value.

<br>

### Repeat the data processing procedure with teosinte files

	$ tail -n +4 teosinte_transposed.txt > teosinte_tran_short.txt
	$ head -n 3 teosinte_transposed.txt > teosinte_header.txt
	$ vi teosinte_header.txt

	$ sort -k1,1 teosinte_tran_short.txt > teosinte_tran_short_sort.txt
	$ join -1 1 -2 1 -t $'\t' -e 'empty' short_snp_all_sort.txt teosinte_tran_short.txt > teosinte_join_full.txt
	$ cat teosinte_header.txt teosinte_join_full.txt > TEOSINTE.txt
	
	$ sort -k2,2n -k3,3n TEOSINTE.txt > TEOSINTE_sort.txt 
	$ awk '{ if ($2 == 1 && $3 !~ /multiple/) { print } }' TEOSINTE_sort.txt > teosinte_chrom_01_IN_headless.txt
	$ cat teosinte_header.txt teosinte_chrom_01_IN_headless.txt > teosinte_chrom_01_increase.txt
	
	$ sort -k2,2n -k3,3nr TEOSINTE.txt > TEOSINTE_reverse.txt
	$ sed 's/?/-/g' TEOSINTE_reverse.txt > TEOSINTE_reverse_sort.txt
	$ awk '{ if ($2 == 1 && $3 !~ /multiple/) { print } }' TEOSINTE_reverse_sort.txt > teosinte_chrom_01_RV_headless.txt
	$ cat teosinte_header.txt teosinte_chrom_01_RV_headless.txt > teosinte_chrom_01_reverse.txt
	
	$ awk '{ if ($2 ~ /unknown/ || $3 ~ /unknown/) { print } }' TEOSINTE_sort.txt > teosinte_chrom_unk_headless.txt
	$ cat teosinte_header.txt teosinte_chrom_unk_headless.txt > teosinte_chrom_unk.txt

	$ awk '{ if ($2 ~ /multiple/ || $3 ~ /multiple/) { print } }' TEOSINTE_sort.txt > teosinte_chrom_mult_headless.txt
	$ cat teosinte_header.txt teosinte_chrom_mult_headless.txt > teosinte_chrom_mult.txt
	
<br>
<br>
<br>

### Move and count the number of input files ready for analysis

	$ mkdir FINAL_INPUT_FILES/
	$ mv maize*_increase.txt maize*_reverse.txt teosinte*_increase.txt teosinte*_reverse.txt *_mult.txt *_unk.txt FINAL_INPUT_FILES/
	$ ls FINAL_INPUT_FILES/ | wc -l
	
> 44

There are 44 files ready for downstream analysis.

<br>

### Quick File Checks

The newly formed files can be previewed and counted to validate the contents.

	$ cut -f 1-5 maize_chrom_01_increase.txt | column -t | head -n 11

|SNP_ID		|Chromosome	|Position	|ZDP_0752a	|ZDP_0793a
| ---			| ---			| ---		| ---	| ---
|SNP_ID		|Chromosome	|Position	|Zmm-LR-ACOM-usa-NM-1_s	|Zmm-LR-ACOM-usa-NM-2
|SNP_ID		|Chromosome	|Position	|ZMMLR	|ZMMLR
|PZA00230.5	|1				|157104	|C/C	|C/C
|PZA00477.11	|1				|13205252	|T/T	|T/T
|PZA00477.9	|1				|4175293	|C/T	|C/C
|PZA00477.10	|1				|4175573	|G/G	|G/G
|PZA00477.5	|1				|4429897	|C/C	|C/C
|PZA00243.27	|1				|4430055	|T/T	|T/T
|PZA00050.9	|1				|4835472	|G/G	|G/G
|PZA00623.2	|1				|4835540	|T/T	|?/?

We can see from this preview of the `maize_chrom_01_increase.txt` file that the SNPs are increasing in order by position and the missing data is encoded by the '?' symbol.

<br>

	$ cut -f 1-5 maize_chrom_01_reverse.txt | column -t | head -n 11
	
|SNP_ID		|Chromosome	|Position	|ZDP_0752a	|ZDP_0793a
| ---			| ---			| ---		| ---	| ---
|SNP_ID		|Chromosome	|Position	|Zmm-LR-ACOM-usa-NM-1_s	|Zmm-LR-ACOM-usa-NM-2
|SNP_ID		|Chromosome	|Position	|ZMMLR	|ZMMLR
|PZB00859.1	|1				|298412984	|G/G	|G/G
|PZA02962.13	|1				|298082627	|G/G	|G/G
|PZA00393.1	|1				|298082534	|C/C	|C/C
|PZA00393.4	|1				|298082504	|G/G	|G/G
|PZA02869.8	|1				|298082468	|T/T	|T/T
|PZA02869.2	|1				|295771152	|A/A	|A/A
|PZD00021.4	|1				|295459549	|A/A	|A/A
|PZD00021.2	|1				|293632755	|C/G	|-/-

We can see from this preview of the `maize_chrom_01_reverse.txt` file that the SNPs are decreasing in order by position and the missing data is encoded by the '-' symbol.

<br>

	$ wc maize*increase.txt maize_chrom_unk.txt maize_chrom_mult.txt

|Lines	|File
| ---	| ---	
|158	|`maize_chrom_01_increase.txt`
|129	|`maize_chrom_02_increase.txt`
|110	|`maize_chrom_03_increase.txt`
|91		|`maize_chrom_04_increase.txt`
|125	|`maize_chrom_05_increase.txt`
|76		|`maize_chrom_06_increase.txt`
|99		|`maize_chrom_07_increase.txt`
|65		|`maize_chrom_08_increase.txt`
|60		|`maize_chrom_09_increase.txt`
|56		|`maize_chrom_10_increase.txt`
|30		|`maize_chrom_unk.txt`
|20		|`maize_chrom_mult.txt`
|1019	|total

	$ wc MAIZE
	
|Lines	|File
| ---	| ---	
|986	|`MAIZE.txt`

Accounting for the 3 header lines in each file we have a net total of 983 lines which matches the net total of the source `MAIZE.txt` file.

> 1019 - (3*12) = 983
 
> 986 - 3 = 983
