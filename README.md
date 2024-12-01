# BIO312-Extra-Credit
Final Repository

## Clone Repository
**You only need to do this ONE time.**

Use the command below to save the GitHub username as a variable named MYGIT.
```
MYGIT="BIO312-Extra-Credit"
```

On the command line, clone the lab repository.
```
git clone https://github.com/s3-khe/BIO312-Extra-Credit.git
```

## 1. BLAST the protein (POLR3A RNA polymerase III subunit A)
**Create a new directory**

You will be conducting a blast search for the protein. Create a folder for the gene BLAST search using the mkdir command:
```
mkdir ~/lab03-$MYGIT/gene
```

Now go to this folder:
```
cd ~/lab03-$MYGIT/gene
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

## 2. Gene family sequence alignment
**Create a new directory**

You will be conducting a sequence alignment for the gene family. Create a folder for the gene family sequence alignment using the mkdir command:
```
mkdir ~/lab04-$MYGIT/gene
```

Now go to this folder:
```
cd ~/lab04-$MYGIT/gene
```

**Align gene family members**

The proteomes are  downloaded and they are all in allprotein.fas. Grab the sequences wanted (their names are in gene.blastp.detail.filtered.out) from allprotein.fas using seqkit.

Here is the seqkit command to obtain the sequences that are in the BLAST output file:
```
seqkit grep --pattern-file ~/lab03-$MYGIT/gene/gene.blastp.detail.filtered.out ~/lab03-$MYGIT/allprotein.fas | seqkit grep -v -p "carpio" > ~/lab04-$MYGIT/gene/gene.homologs.fas
```

**Perform a global multiple sequence alignment in muscle**

Align all of the sequences with each other along their entire length. The resulting alignment will be a hypothesis about which positions are homologous to each other among the gene proteins, and which positions contain insertions or deletions, with respect to the other sequences.

Use the following command to make a multiple sequence alignment using muscle:
```
muscle -align ~/lab04-$MYGIT/gene/gene.homologs.fas -output ~/lab04-$MYGIT/gene/gene.homologs.al.fas
```

**View the alignment as a pdf and save it**

Download the pdf viewer by "tomoki1207" extension.

View the alignment in alv:
```
alv -kli ~/lab04-$MYGIT/gene/gene.homologs.al.fas | less -RS
```

Try the "majority" option
```
alv -kli --majority ~/lab04-$MYGIT/gene/gene.homologs.al.fas | less -RS
```

Use the R package msa and a script to print your alignment to a large pdf file that can be inspected with ease. Run this Rscript for printing the alignment:
```
 Rscript --vanilla ~/lab04-$MYGIT/plotMSA.R  ~/lab04-$MYGIT/globins/globins.homologs.al.fas
```

If the above command doesn't work, use the following alternative command:
```
alv -kil -w 100 ~/lab04-$MYGIT/gene/gene.homologs.al.fas | aha > ~/lab04-$MYGIT/gene/gene.homologs.al.html

a2ps -r --columns=1 ~/lab04-$MYGIT/gene/gene.homologs.al.html -o ~/lab04-$MYGIT/gene/gene.homologs.al.ps

ps2pdf ~/lab04-$MYGIT/gene/gene.homologs.al.ps ~/lab04-$MYGIT/gene/gene.homologs.al.pdf
```

**Information about the alignment**

Calculate the width (length) of the alignment
```
alignbuddy -al ~/lab04-$MYGIT/gene/gene.homologs.al.fas
```

Calculate the length of the alignment after removing any column with gaps using the following command:
```
alignbuddy -trm all ~/lab04-$MYGIT/gene/gene.homologs.al.fas | alignbuddy -al
```

Calculate the length of the alignment after removing invariant (completely conserved) positions using the following command:
```
alignbuddy -dinv 'ambig' ~/lab04-$MYGIT/gene/gene.homologs.al.fas | alignbuddy -al
```

**Calculate the average percent identity**

Calculate the average percent identity in the alignment using t_coffee:
```
t_coffee -other_pg seq_reformat -in ~/lab04-$MYGIT/gene/gene.homologs.al.fas -output sim
```

Repeat calculating the average percent identity using alignbuddy:
```
alignbuddy -pi ~/lab04-$MYGIT/gene/gene.homologs.al.fas | awk ' (NR>2) { for (i=2;i<=NF ;i++){ sum+=$i;num++} } END{ print(100*sum/num) } '
```

## 3. Gene Family Phylogeny using IQ-TREE
**Create a new directory**

You will be creating a phylogeny tree for the gene family using the program IQ_TREE. Create a folder for the gene family phylogeny tree using the mkdir command:
```
mkdir ~/lab05-$MYGIT/gene
```

Now go to this folder:
```
cd ~/lab05-$MYGIT/gene
```

**Constructing a Phylogenetic Tree for gene Homologs from Sequence Data**

Use the software IQ-TREE to infer the optimal phylogenetic tree based on a sequence alignment. 

The first step is to remove any sequence that contains a duplicate label tag by using the following command:
```
sed 's/ /_/g' ~/lab04-$MYGIT/gene/gene.homologs.al.fas | seqkit grep -v -r -p "dupelabel" > ~/lab05-$MYGIT/gene/gene.homologsf.al.fas
```

The following command will use IQ-TREE to find the maximum likehood tree estimate.
```
iqtree -s ~/lab05-$MYGIT/gene/gene.homologsf.al.fas -bb 1000 -nt 2
```

**Create the unrooted tree**

The command below will display an unrooted tree of the gene family in a text graphic version.
```
nw_display ~/lab05-$MYGIT/gene/gene.homologsf.al.fas.treefile
```

To look at the unrooted tree in a graphical display, use the command below: 
```
Rscript --vanilla ~/lab05-$MYGIT/plotUnrooted.R ~/lab05-$MYGIT/gene/gene.homologsf.al.fas.treefile ~/lab05-$MYGIT/gene/gene.homologsf.al.fas.treefile.pdf 0.4 15
```

**Midpoint rooting**

The following command will create a tree in a text graphic version that will use a type of rooting called midpoint. In this tree, the root will be halfway along the longest branch on the tree.
```
gotree reroot midpoint -i ~/lab05-$MYGIT/gene/gene.homologsf.al.fas.treefile -o ~/lab05-$MYGIT/gene/gene.homologsf.al.mid.treefile
```

Use the command below to view the rooted tree that was just created.
```
nw_order -c n ~/lab05-$MYGIT/gene/gene.homologsf.al.mid.treefile | nw_display -
```

To look at the rooted tree in a graphical display, use the command below: 
```
nw_order -c n ~/lab05-$MYGIT/gene/gene.homologsf.al.mid.treefile | nw_display -w 1000 -b 'opacity:0' -s > ~/lab05-$MYGIT/gene/gene.homologsf.al.mid.treefile.svg -
```

Convert this svg to a pdf using the following command:
```
convert ~/lab05-$MYGIT/gene/gene.homologsf.al.mid.treefile.svg ~/lab05-$MYGIT/gene/gene.homologsf.al.mid.treefile.pdf
```

**Branch lengths**

To help look at short branches in a phylogeny, viewing a cladogram would be better than viewing a phylogram. Use the following command:
```
nw_order -c n ~/lab05-$MYGIT/gene/gene.homologsf.al.mid.treefile | nw_topology - | nw_display -s -w 1000 > ~/lab05-$MYGIT/gene/gene.homologsf.al.midCl.treefile.svg -
convert ~/lab05-$MYGIT/gene/gene.homologsf.al.midCl.treefile.svg ~/lab05-$MYGIT/gene/gene.homologsf.al.midCl.treefile.pdf
```

## 4. Reconciling a Gene and Species Tree
**You only need to do this ONE time.**
Install the following program using this command:
```
mamba create -n my_python27_env python=2.7
conda activate my_python27_env
mamba install -y ete3
```

**Create a new directory**

You will be reconciling a gene and species tree using the program Notung. Create a folder for this reconciliation using the mkdir command:
```
mkdir ~/lab06-$MYGIT/gene
```

Now go to this folder:
```
cd ~/lab06-$MYGIT/gene
```

Make a copy of the gene tree from lab 5 gene family folder into lab 6 gene family folder using the following command:
```
cp ~/lab05-$MYGIT/gene/gene.homologsf.al.mid.treefile ~/lab06-$MYGIT/gene/gene.homologsf.al.mid.treefile
```

**Reconciling gene and species tree**
Use the following command to perform the reconciliation:
```
java -jar ~/tools/Notung-3.0_24-beta/Notung-3.0_24-beta.jar -s ~/lab05-$MYGIT/species.tre -g ~/lab06-$MYGIT/gene/gene.homologsf.al.mid.treefile --reconcile --speciestag prefix --savepng --events --outputdir ~/lab06-$MYGIT/gene/
```

View the node names assigned by Notung using the command below:
```
grep NOTUNG-SPECIES-TREE ~/lab06-$MYGIT/gene/gene.homologsf.al.mid.treefile.rec.ntg | sed -e "s/^[&&NOTUNG-SPECIES-TREE//" -e "s/]/;/" | nw_display -
```

**Generate a RecPhyloXML object**
Use the following command to generate a RecPhyloXML object:
```
python2.7 ~/tools/recPhyloXML/python/NOTUNGtoRecPhyloXML.py -g ~/lab06-$MYGIT/gene/gene.homologsf.al.mid.treefile.rec.ntg --include.species
```

If the command above does not work (shows the error: bash: python2.7: command not found), run the command below:
```
conda activate my_python27_env
```

**View the gene-within-species tree via thirdkind**
To view the generated gene-reconciliation-within species tree with thirdkind, use the following command:
```
thirdkind -Iie -D 40 -f ~/lab06-$MYGIT/gene/gene.homologsf.al.mid.treefile.rec.ntg.xml -o ~/lab06-$MYGIT/gene/gene.homologsf.al.mid.treefile.rec.svg
```

Then convert this svg to a pdf for easy viewing:
```
convert -density 150 ~/lab06-$MYGIT/gene/gene.homologsf.al.mid.treefile.rec.svg ~/lab06-$MYGIT/gene/gene.homologsf.al.mid.treefile.rec.pdf
```

## 5. Protein Domain Prediction
**Create a new directory**

You will be identifying the protein domain of the gene using the programs RPS-BLAST and Pfam. Create a folder for this using the mkdir command:
```
mkdir ~/lab08-$MYGIT/gene
```

Now go to this folder:
```
cd ~/lab08-$MYGIT/gene
```

Make a copy of the raw unaligned sequence using sed's substiture command, which substitutes any instance of an asterisk with nothing.
```
sed 's/*//' ~/lab04-$MYGIT/gene/gene.homologs.fas > ~/lab08-$MYGIT/gene/gene.homologs.fas
```

**Run RPS-BLAST**
Run RPS-BLAST using the command below:
```
rpsblast -query ~/lab08-$MYGIT/gene/gene.homologs.fas -db ~/data/Pfam/Pfam -out ~/lab08-$MYGIT/gene/gene.rps-blast.out -outfmt "6 qseqid qlen qstart qend evalue stitle" -evalue .0000000001
```

**Plot the predicterd Pfam domains**
To plot the predicted Pfam domains on a phylogeny, first copy the gene tree that was stored in the lab 5 directory by using the following command:
```
cp ~/lab05-$MYGIT/gene/gene.homologsf.al.mid.treefile ~/lab08-$MYGIT/gene
```

Then use the Rscript program to run a script that was written in R by Dr. Rest to plot the Pfam domain predictions from RPS-BLAST next to their cognate protein on the phylogeny. The command below will run the script:
```
Rscript --vanilla ~/lab08-$MYGIT/plotTreeAndDomains.r ~/lab08-$MYGIT/gene/gene.homologsf.al.mid.treefile ~/lab08-$MYGIT/gene/gene.rps-blast.out ~/lab08-$MYGIT/gene/gene.tree.rps.pdf
```

**Look at plotted predicted Pfam domains**
Now that the predicted Pfam domains have been plotted, its details can be viewed using the command below:
```
mlr --inidx --ifs "\t" --opprint cat ~/lab08-$MYGIT/gene/gene.rps-blast.out | tail -n +2 | less -S
```

To find which Pfam domain does not have annotations, use the following command:
```
cut -f 1 ~/lab08-$MYGIT/gene/gene.rps-blast.out | sort | uniq -c
```

To find which Pfam domain is most commonly found, use the following command:
```
cut -f 6 ~/lab08-$MYGIT/gene/gene.rps-blast.out | sort | uniq -c
``` 

To find which Pfam domains have the shortest and longest annotated protein domains, use the following command:
```
awk '{a=$4-$3;print $1,'\t',a;}' ~/lab08-$MYGIT/gene/gene.rps-blast.out |  sort  -k2nr
```

To find which Pfam domain annotation has the best e-value, use the following command:
```
cut -f 1,5 -d $'\t' ~/lab08-$MYGIT/gene/gene.rps-blast.out
```
