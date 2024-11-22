# BIO312-Extra-Credit
Final Repository

## Clone Repository
**You only need to do this ONE time.**

On the command line, clone the lab 3 repository.
```
git clone https://github.com/s3-khe/BIO312-Extra-Credit.git
```

## 1. BLAST the protein (POLR3A RNA polymerase III subunit A)
**Create a new directory**

You will be conducting a blast search for the protein. Create a folder for the globin BLAST search using the mkdir command:
```
mkdir ~/BIO312-Extra-Credit/gene
```

Now go to this folder:
```
cd ~/BIO312-Extra-Credit/gene
```

**Download the query protein**

Download the query protein using the following command:
```
ncbi-acc-download -F fasta -m protein "NP_008986.2" 
```

**Perform the BLAST search**

Use this comand to perform a blast search using the query protein:
```
blastp -db ../allprotein.fas -query NP_008986.2.fa -outfmt 0 -max_hsps 1 -o ut gene.blastp.typical.out less gene.blastp.typical.out 
```

**Perform a BLAST search, and request tabular output**

This command creates a more detailed and easier-to-process output of the same analysis. The -outfmt flag specifies a particular output format that will be useful for our analysis. 

Type in the following command:
```
blastp -db ../allprotein.fas -query NP_008986.2.fa -outfmt "6 sseqid pident length mismatch gapopen evalue bitscore pident stitle" -max_hsps 1 -out gene.blastp.detail.out
```

Take a look at the output in gene.blastp.detail.out using the less -S command:
```
less -S gene.blastp.detail.out
```

Count the number of hits are in the file using the following command:
```
grep -c H.sapiens gene.blastp.detail.out
```

**Filtering the BLAST output for high-scoring putative homologs**

Choose which putative homologs to include by only including high-scoring matches for analysis. Let's require the e-value to be less than 1e-30. Use this command you to filter our output file to satisfy this requirement:
```
awk '{if ($6< 1e-30)print $1 }' gene.blastp.detail.out > gene.blastp.detail.filtered.out
```

Use the wc command to count the total number of hits in the BLAST results after the filter.
```
wc -l gene.blastp.detail.filtered.out
```

**How many paralogs are in each species?**

To find the number of paralogs are in each species, use the following command:
```
grep -o -E "^[A-Z].[a-z]+" gene.blastp.detail.filtered.out | sort | uniq -c
```
