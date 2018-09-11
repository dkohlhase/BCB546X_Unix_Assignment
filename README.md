# BCB 546X - Unix Assignment
### Due: September 21 by 17:00 
### Daniel Kohlhase
[Data Inspection](## **Data Inspection**)

[Data Processing](## **Data Processing**)

<br>
## **Data Inspection**
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
## Data **Processing**
### Extract maize data from genotype file 

First we need the column headers.

	$ head -n 1 fang_et_al_genotypes.txt > maize.txt

We can extract maize data in one fell swoop.
	
	$ grep -e "ZMMIL" -e "ZMMLR" -e "ZMMMR" fang_et_al_genotypes.txt >> maize.txt
	$ wc maize.txt

|Lines		|Words		|Characters	|File	
| --- 		| --- 		| --- 			| ---
|1574		|1551964	|6250961		|`maize.txt`

<br>
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

<br>
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
|968		|1551964	|6250961		|`maize_transposed.txt`

	$ awk -f transpose.awk teosinte.txt > teosinte_transposed.txt
	$ wc teosinte_transposed.txt

|Lines		|Words		|Characters	|File	
| --- 		| --- 		| --- 			| ---
|968		|962336	|3884185		|`teosinte_transposed.txt`

The number of lines has changed in both files and now match. The number of words and characters remained the same for the respective files, which is what we would expect because all we did was rearrange (transpose) the file.

### Create header and information files from the genotype file

The number of lines is different for the genotype and snp files (986 vs 984). The genotype files have an additional two lines containing additional sample information. 

Create two new files. 

The first file is the transposed genotype files **minus the top two lines** (*_short.txt). 

	$ tail -n +4 maize_transposed.txt > maize_tran_short.txt

The second file is a header file which is **just the top two lines** of the transposed genotype files (*_joined.txt), .

	$ head -n 3 maize_transposed.txt > maize_header.txt

Manually add labels for the SNP_ID, Chromosome, Position in the first three columns of the header file.
	
	$ vi maize_header.txt

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

### Combine the SNP and genotype information by columns and append to header file

	$ paste short_snp_all_sort.txt maize_tran_short.txt > maize_join_cut.txt
	

**^SHOULD** be the `join` function:
	
	$ join -t "\t" -1 1 -2 1 -e '' short_snp_all_sort.txt maize_tran_short_sort.txt > TEST.txt
	
<br>
	
Add headers to the file with the combined SNP and genotype information.
	 
	$ cat maize_header.txt maize_join_cut.txt > MAIZE.txt
	
### Extract data for input files

10 files (1 for each chromosome) with SNPs ordered based on increasing position values and with missing data encoded by this symbol: ?

	$ sort -k2,2n -k3,3n MAIZE.txt > MAIZE_sort.txt 
	$ awk '{ if ($2 == 1) { print } }' MAIZE_sort.txt > maize_chrom_01_IN_headless.txt
	$ cat maize_header.txt maize_chrom_01_IN_headless.txt > maize_chrom_01_increase.txt

The 'awk' command followed by appending the subset to the maize header file is repeated for each chromosome.

<br>	

10 files (1 for each chromosome) with SNPs ordered based on decreasing position values and with
missing data encoded by this symbol: -

	$ sort -k2,2n -k3,3nr MAIZE.txt > MAIZE_reverse.txt
	$ sed 's/?/-/g' MAIZE_reverse.txt > MAIZE_reverse_sort.txt
	$ awk '{ if ($2 == 1) { print } }' MAIZE_reverse_sort.txt > maize_chrom_01_RV_headless.txt
	$ cat maize_header.txt maize_chrom_01_RV_headless.txt > maize_chrom_01_reverse.txt

The addition of the 'sed' program substitutes -/- for ?/? to identify missing information.

<br>

1 file with all SNPs with unknown positions in the genome (these need not be ordered in any particular way)

	$ awk '{ if ($2 ~ /unknown/) { print } }' MAIZE_sort.txt > maize_chrom_unk_headless.txt
	$ cat maize_header.txt maize_chrom_unk_headless.txt > maize_chrom_unk.txt
	
1 file with all SNPs with multiple positions in the genome (these need not be ordered in any particular way)
	
	$ awk '{ if ($2 ~ /multiple/) { print } }' MAIZE_sort.txt > maize_chrom_mult_headless.txt
	$ cat maize_header.txt maize_chrom_mult_headless.txt > maize_chrom_mult.txt

The same 'awk' command is used to extract the unknown and multiple SNP information except adapted to search for an alphanumeric string rather than a value.

<br>

### Repeat the data processing procedure with teosinte files

	$ tail -n +4 teosinte_transposed.txt > teosinte_tran_short.txt
	$ head -n 3 teosinte_transposed.txt > teosinte_header.txt
	$ vi teosinte_header.txt

	$ sort -k1,1 teosinte_tran_short.txt > teosinte_tran_short_sort.txt
	
	$ paste short_snp_all_sort.txt teosinte_tran_short.txt > teosinte_join_cut.txt

	$ cat teosinte_header.txt teosinte_join_cut.txt > TEOSINTE.txt
	
	$ sort -k2,2n -k3,3n TEOSINTE.txt > TEOSINTE_sort.txt 
	$ awk '{ if ($2 == 1) { print } }' TEOSINTE_sort.txt > teosinte_chrom_01_IN_headless.txt
	$ cat teosinte_header.txt teosinte_chrom_01_IN_headless.txt > teosinte_chrom_01_increase.txt
	
	$ sort -k2,2n -k3,3nr TEOSINTE.txt > TEOSINTE_reverse.txt
	$ sed 's/?/-/g' TEOSINTE_reverse.txt > TEOSINTE_reverse_sort.txt
	$ awk '{ if ($2 == 1) { print } }' TEOSINTE_reverse_sort.txt > teosinte_chrom_01_RV_headless.txt
	$ cat teosinte_header.txt teosinte_chrom_01_RV_headless.txt > teosinte_chrom_01_reverse.txt
	
	$ awk '{ if ($2 ~ /unknown/) { print } }' TEOSINTE_sort.txt > teosinte_chrom_unk_headless.txt
	$ cat teosinte_header.txt teosinte_chrom_unk_headless.txt > teosinte_chrom_unk.txt

	$ awk '{ if ($2 ~ /multiple/) { print } }' TEOSINTE_sort.txt > teosinte_chrom_mult_headless.txt
	$ cat teosinte_header.txt teosinte_chrom_mult_headless.txt > teosinte_chrom_mult.txt
	
<br>

### Move and count the number of input files ready for analysis

	$ mkdir FINAL_INPUT_FILES/
	$ mv maize*_increase.txt maize*_reverse.txt teosinte*_increase.txt teosinte*_reverse.txt *_mult.txt *_unk.txt
	$ ls FINAL_INPUT_FILES/ | wc -l
	
> 44

There are 44 files ready for downstream analysis.
