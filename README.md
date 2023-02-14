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
- Runs a quality assessment check on all the whole-genome sequences uploaded to the online server using the program CheckM.
- Returns quality assessment files include the percentage of completeness and the degree of contamination for each given genome.
- The output files are saved in a directory named "output".
~~~

## qa_filtering.py
**Local**: *Used after importing the "output" directory locally.*
~~~
- Reads the quality assessment output files generated by CheckM.
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


## first_tree.py
**Local**: *Used after separating the acetic bacteria from the lactic bacteria and filtering out contaminated and incomplete genomes.*
~~~
- Retrieves the percentage of completeness of all genomes from the "bacteria log v2.xlsx" Excel sheet (the updated version of "bacteria_log.xlsx").
- For each species of interest (acetic or lactic), finds the file corresponding to the most complete genome of that species.
- Creates a new directory ("Acetic_unique"/"Lactic_unique") containing only the most complete genome per species.
~~~

This directory is then exported to the server so that **Prokka** can be used to annotate all the selected genomes.

## prokka.sh
**Mjolnir**: *Used after exporting the "Acetic_unique"/"Lactic_unique" directory (containing only the most complete genomes) to the server.*
~~~
- Uses the rapid prokaryotic genome annotation program Prokka to annotate the genomes, this will make it a lot easier to align them later on.
- Returns multiple files but the ".gff" files are the ones that interest us.
~~~

Once the genomes have been annotated, they can be aligned using **Roary**.

~~~
roary -e --mafft -p 8 *.gff
~~~
*-e --mafft* aligns the core genes using the tool MAFFT.
*-p 8* uses 8 threads.

**What does Roary do?**
- Converts coding sequences into protein sequences.
- Cluster these protein sequences by several methods.
- Further refines clusters into orthologous genes.
- For each sample, determines if gene is present/absent: produces "gene_presence_absence.csv".
- Uses this gene p/a information to build a tree, using FastTree: produces "accessory_binary_genes.fa.newick".
- Overall, calculates number of genes that are shared, and unique: produces "summary_statistics.txt".
- Aligns the core genes (if option used, as above) for downstream analyses.


## roary.sh
**Mjolnir**: *Used after annotating the genomes located in the "Acetic_unique"/"Lactic_unique" directory with Prokka.*
~~~
- Uses the pan-genome pipeline program Roary to form a pan-genome alignent from the genomes using the ".gff" files.
- The alignment is executed using Mafft.
- Returns many files but the "core_gene_alignment.aln" file is the one that interests us.
~~~

Once the genomes have been aligned, we can construct the tree using **FastTree**.


## tree.sh
**Mjolnir**: *Used after constructing the pan-genome alignment with Roary.*
~~~
- Generates the tree file in Newick format ("tree.newick") from the core alignement file ("core_gene_alignment.aln") using FastTree.
- Single-line commmand: 
  FastTree -nt -gtr output_with_alignment/core_gene_alignment.aln > tree.newick
~~~

## Visualisation
**Local**: *Single line of code to visualise the tree.*
~~~
python3 roary_plots.py tree.newick gene_presence_absence.csv
~~~
