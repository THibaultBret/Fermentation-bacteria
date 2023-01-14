# Fermentation-bacteria

The scripts used as part of this project are here described in order:

## initialiser.py
**Local**: *Used after downloading whole-genomes sequences from NCBI.*
 ~~~
- Prepares the working directory.
- Fills the bacteria log dataframe using the "data_summary.tsv" file present in the folder downloaded from NCBI.
- The log is filled with informations such as the organism qualifier, taxonomic ID, assembly name/accession, size, gene count, BioProject ID, BioSample ID, etc.
- Removes unnecessary files.
- Moves all the genome sequences to the bacterium-specific folder (removes unnecessary nested folders).
- Saves the full bacteria log data as an Excel sheet --> "bacteria_log.xlsx"
~~~


## qa.sh
**Mjolnir**: *Used after exporting the whole-genome sequence files.*
~~~
- Runs a quality assessment check on all the whole-genome sequences uploaded to the online server using the program **CheckM**.
- Returns quality assessment files include the percentage of completeness and the degree of contamination for each given genome.
- The output files are saved in a directory named "output".
~~~

## qa_filtering.py
**Local**: *Used after importing the "output" directory locally.*
~~~
- Reads the quality assessment output files generated by **CheckM**.
- Converts all the qa data into a Pandas dataframe that is then converted into an Excel sheet --> "total_qa_df.xlsx".

- Reads the "total_qa_df.xlsx" Excel sheet.
- Based on the completeness and the percentage of contamination of each genome, given the predetermined threshold parameters (completeness threshold = 0.9; contamination threshold = 0.05), filters out genomes that are not fit for the analysis. 
- This is done by creating a new directory called "Genomes_filtered" where only the valid whole-genome sequences are present.
~~~


## acetic_or_lactic.py
**Mjolnir**: *Used after exporting the "Genomes_filtered" directory to the Mjolnir server.*
~~~
- Writes a file ("sort_file.txt") that lists all the genomes and indicates whether each genome is part of the Lactic or the Acetic bacteria family.

- Reads the "sort_file.txt" file that assigns every genome to either the Lactic or the Acetic bacteria family.
- Correspondingly moves the genome sequence files in the "Acetic" folder or the "Lactic" folder. 
- That way, lactic and acetic bacteria can be treated separately for later steps of the analysis.
~~~
