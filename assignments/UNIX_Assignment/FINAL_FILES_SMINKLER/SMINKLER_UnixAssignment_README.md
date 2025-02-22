UNIX Assignment 
Sarah Minkler

Data Inspection

By inspecting this fang_et_al_genotypes.txt, I learned 
•	The file size is 11 Mb
$ ls -lh fang_et_al_genotypes.txt
•	Lines: 2783; words; 2744038; bites(characters): 11051939
$ wc fang_et_al_genotypes.txt
•	986 columns are present in file
$ awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt
By inspecting this data file, I learned snp_position.txt (same code as used above, but file name changed)…
•	File size: 81Kb
•	Lines: 984; words: 13198; bites: 82763
•	15 columns in this file
Data Processing
Maize Data
•	Gathering column headers into new file ‘maize.txt’
$ head -n 1 fang_et_al_genotypes.txt > maize.txt
•	Putting ZMMIL, ZMMLR, ZMMMR into maize file
$ grep -e "ZMMIL" -e "ZMMLR" -e "ZMMMR" fang_et_al_genotypes.txt >> maize.txt
•	Check for entries in each column and counting the number of entries for each column
$ cut -f 3 maize.txt | sort | uniq -c
      1 Group
    290 ZMMIL
   1256 ZMMLR
     27 ZMMMR
Teosinte Data
•	Coding same as above to extract column headers and checking entries for each column
$ grep  -e "ZMPBA" -e "ZMPIL" -e "ZMPJA" fang_et_al_genotypes.txt >> teosinte.txt
o	New file for Teosinte data called ‘teosinte.txt’
•	Double checking for three unique columns (same code as above)
1 Group
900 ZMPBA
41 ZMPIL
34 ZMPJA

Transposing files together (used maize for example, but also done with teosinte data)
$ awk -f transpose.awk maize.txt > transposed_maize_genotypes.txt
$ awk -f transpose.awk teosinte.txt > transposed_teosinte_genotypes.txt
$ wc transposed_maize_genotypes.txt transposed_teosinte_genotypes.txt 
     986  1551964  6250961 transposed_maize_genotypes.txt
     986  962336  3884185 transposed_teosinte_genotypes.txt
•	I checked to make sure the number of lines in each file matched before moving on (above). 
•	The number of lines is different for the genotype and snp files; 986 vs 984, so I created two new tiles to fix this. 
•	The first file is the transposed genotype files minus the top two lines labeled short.txt. The header file contains just the top two lines of the transposed genotype files (joined.txt). Then I manually added labels (SNP_ID, chromosome, Position) using vi.
$ tail -n +4 transposed_maize_genotypes.txt > maize_transposed_short.txt
$ head -n 3 transposed_maize_genotypes.txt > maize_header.txt
$ vi maize_header.txt
•	I then extracted and rearranged SNP data to format them in the correct orientation.
$ cut -f 1 snp_position.txt > short_snp.txt
$ cut -f 3 snp_position.txt > short_snp_chromo.txt
$ cut -f 4 snp_position.txt > short_snp_posit.txt
•	Pasting files together after extracting desired data
$ paste short_snp.txt short_snp_chromo.txt > short_snp_plus_chromo.txt
$ paste short_snp_plus_chromo.txt short_snp_posit.txt > short_snp_all.txt
•	Sorting files by SNP before joining them together with genotype files
$ sort -k1,1 short_snp_all.txt > short_snp_all_sort.txt 
$ sort -k1,1 maize_transposed_short.txt > maize_trans_short_sorted.txt

•	To combine SNP and genotype files together, I used ‘join’ and then added in the header using ‘cat’
$ join -1 1 -2 1 -t $'\t' -e 'empty' short_snp_all_sort.txt maize_transposed_short.txt > maize_join_full.txt
$ cat maize_header.txt maize_join_full.txt > Maize.txt

Data extraction for final data files
10 files (1 for each chromosome) with SNPs ordered based on increasing position values and with missing data encoded by this symbol: ?
•	Showing for maize data, but also performed in teosinte data. The code I used for chromosome 1 can be seen below, but this same code was repeated for all 10 chromosomes. 
$ sort -k2,2n -k3,3n Maize.txt > Maize_sort.txt 
$ awk '{ if ($2 == 1 && $3 !~ /multiple/) { print } }' Maize_sort.txt > maize_chromo_01_inc_headless.txt
$ cat maize_header.txt maize_chromo_01_inc_headless.txt > maize_chromo_01_increasing.txt

10 files (1 for each chromosome) with SNPs ordered based on decreasing position values and with missing data encoded by this symbol: -
•	I used the ‘sed’ program to substitute – for ?.
$ sort -k2,2n -k3,3nr Maize.txt > Maize_decreasing.txt
$ sed 's/?/-/g' Maize_decreasing.txt > Maize_decreasing_sorted.txt
$ awk '{ if ($2 == 1 && $3 !~ /multiple/) { print } }' Maize_decreasing_sort.txt > maize_chromo_01_dec_headless.txt
$ cat maize_header.txt maize_chromo_01_dec_headless.txt > maize_chrom_01_decreasing.txt

1 file with all SNPs with unknown positions in the genome (these need not be ordered in any particular way)
$ awk '{ if ($2 ~ /unknown/ || $3 ~ /unknown/) { print } }' Maize_sort.txt > maize_chromosome_unknown_headless.txt
$ cat maize_header.txt maize_chrom_unk_headless.txt > maize_chromosome_unknown.txt

1 file with all SNPs with multiple positions in the genome (these need not be ordered in any particular way)
$ awk '{ if ($2 ~ /multiple/ || $3 ~ /multiple/) { print } }' Maize_sort.txt > maize_chromosome_multi_headless.txt
$ cat maize_header.txt maize_chromosome_multi_headless.txt > maize_chromosome_multi.txt

These same steps were repeated for teosinte files.
$ tail -n +4 transposed_teosinte_genotypes.txt > teosinte_transposed_short.txt
$ head -n 3 transposed_teosinte_genotypes.txt > teosinte_header.txt
$ vi teosinte_header.txt

$ sort -k1,1 teosinte_transposed_short.txt > teosinte_trans_short_sorted.txt
$ join -1 1 -2 1 -t $'\t' -e 'empty' short_snp_all_sort.txt teosinte_transposed_short.txt > teosinte_join_full.txt
$ cat teosinte_header.txt teosinte_join_full.txt > Teosinte.txt

$ sort -k2,2n -k3,3n Teosinte.txt > Teosinte_sort.txt 
$ awk '{ if ($2 == 1 && $3 !~ /multiple/) { print } }' Teosinte_sort.txt > teosinte_chrom_01_IN_headless.txt
$ cat teosinte_header.txt teosinte_chromo_01_inc_headless.txt > teosinte_chromo_01_increasing.txt

$ sort -k2,2n -k3,3nr Teosinte.txt > Teosinte_decreasing.txt
$ sed 's/?/-/g' Teosinte_decreasing.txt > Teosinte_decreasing_sorted.txt
$ awk '{ if ($2 == 1 && $3 !~ /multiple/) { print } }' Teosinte_decreasing_sorted.txt > teosinte_chromo_01_dec_headless.txt
$ cat teosinte_header.txt teosinte_chromo_01_dec_headless.txt > teosinte_chromo_01_decreasing.txt

$ awk '{ if ($2 ~ /unknown/ || $3 ~ /unknown/) { print } }' Teosinte_sort.txt > teosinte_chromosome_unknown_headless.txt
$ cat teosinte_header.txt teosinte_chromosome_unknown_headless.txt > teosinte_chromosome_unknown.txt

$ awk '{ if ($2 ~ /multiple/ || $3 ~ /multiple/) { print } }' Teosinte_sort.txt > teosinte_chromosome_multi_headless.txt
$ cat teosinte_header.txt teosinte_chromosome_multi_headless.txt > teosinte_chromosome_multi.txt

Finally, I created a directory of all of my final files. 
$ mkdir FINAL_FILES_SMINKLER/
$ mv maize*_increasing.txt maize*_decreasing.txt teosinte*_increasing.txt teosinte*_decreasing.txt *_multi.txt *_unknown.txt FINAL_FILES_SMINKLER/

