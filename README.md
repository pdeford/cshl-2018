# cshl-2018

To produce the Unique Genes Annotation file for the tutorial, go to the [UCSC Table Browser](https://genome.ucsc.edu/cgi-bin/hgTables) and use the following options:

- clade: Mammal
- genome: Human
- assembly: Dec. 2013 (GRCh38/hg38)
- group: Genes and Gene Predictions
- track: NCBI RefSeq
- table: RefSeq Curated (ncbiRefSeqCurated)
- region: genome
- output format: all fields from selected table
- output file: ncbi_genes.txt

Then in the `bash` shell run:

```awk 'BEGIN{OFS="\t"}{if(!_[$13]++){print $3, $7, $8, $13, $12, $4}}' ncbi_genes.txt | tail -n+2 > Unique_refSeq_Genes.bed```
