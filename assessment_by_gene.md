# Assessment by gene
### Taylor Dunivin

__Goals:__
* Be certain that when considering nr database, the gene of interest is still the top hit
* Add to confidence in assembly assessments
* Determine whether/ how to troubleshoot low quality results from specific genes

### Table of contents
* [Setting up workflow](https://github.com/ShadeLab/Xander_arsenic/blob/master/assessment_by_gene.md#setting-up-workflow)
* [blast.qsub](https://github.com/ShadeLab/Xander_arsenic/blob/master/assessment_by_gene.md#blast.qsub)
* [As resistance gene analysis notes](https://github.com/ShadeLab/Xander_arsenic/blob/master/assessment_by_gene.md#as-resistance-gene-analysis-notes)
* [Antibiotic resistance gene analysis notes](https://github.com/ShadeLab/Xander_arsenic/blob/master/assessment_by_gene.md#antibiotic-resistance-gene-analysis-notes)

### __Setting up workflow:__
1. Access HPCC's nr database
```
export BLASTDB=/mnt/research/common-data/Bio/blastdb:$BLASTDB
```

2. Merge all protein fasta files to one `final_prot.fasta`
```
cat *_final_prot.fasta >final_prot.fasta
```

3. Blast nr database: use protein blast `-p blastp`, limit outputs to the two top hits `-b 2 -v 2`, e-values greater than E-6 `e 1e-6`, run with 8 threads `-a 8`
```
blastall -d nr -i final_prot.fasta -p blastp -o blast.txt -b 2 -v 2 -e 1e-6 -a 8
```

4. Make summary file that only shows sequence descriptors (typically gene name)
```
grep '^>' blast.txt > descriptor.blast.txt
```

### __blast.qsub file__
Use .qsub script below and replace `gene` with your gene of interest to run.
```
#!/bin/bash -login
 
### define resources needed:
### walltime - how long you expect the job to run
#PBS -l walltime=04:00:00
 
### nodes:ppn - how many nodes & cores per node (ppn) that you require
#PBS -l nodes=1:ppn=8
 
### mem: amount of memory that the job will need
#PBS -l mem=50gb
 
### you can give your job a name for easier identification
#PBS -N blast.GENE
 
### load necessary modules, e.g.
module load BLAST/2.2.26
 
### change to the working directory where your code is located
cd /mnt/research/ShadeLab/WorkingSpace/Dunivin/xander/blast/GENE
 
### call your executable
export BLASTDB=/mnt/research/common-data/Bio/blastdb:$BLASTDB
cat *_final_prot.fasta >final_prot.fasta
blastall -d nr -i final_prot.fasta -p blastp -o blast.txt -b 2 -v 2 -e 1e-6 -a 8
grep '^>' blast.txt > descriptor.blast.txt
```

### As resistance gene analysis notes
(also includes rplB)
* acr3  
  * 10 hypothetical protein (0.9%)
  * 13 bile acid:sodium symporter (1.2%) 
  * 2 conserved membrane protein of unknown function (0.2%)
  * 1051 total
  * Bile acid:sodium symporter has an alternative name of arsenite resistance protein (OK)
* aioA
  * a couple say "arsenate reductase (azurin)" but these have alt name of arsenite oxidase large subunit (OK)
  * all others are good! 
  * 90 total
* arsB       
  * 7 total
  * all perfect hits!
* arsC_thio  
  * many "protein tyrosine phosphatase" but alt name is arsenate reductase (OK)
  * hypothetical proteins look good too 
* arsC_glut
  * 53 hypothetical (4.9%)
  * 1075 total
  * all others are arsenate reductase!
* arsM
  * 1431 hypothetical protein hits (41.9%)
  * 357 type 11 methyltransferase (10.5%)
  * 24 conserved protein of unknown function (0.7%)
  * 71 putative methyltransferase (2.1%)
  * 70 methyltransferase (2.0%)
  * 53 methylase involved in ubiquinone/menaquinone biosynthesis (1.5%)
  * 27 ubiE/COQ5 methyltransferase (0.8%)
  * 53 BFD-like (2Fe-2S) protein (1.6%)
  * 3415 total
  * There are much too many non-specific hits for arsM
  * To attempt to combat this, I will re-run arsM with more stringent bit score 60 instead of 50
* rplB  

### Antibiotic resistance gene analysis notes
* AAC3-Ia
  * only one hit total (1 from C06)
  * Hit is labeled "acetyltransferase" 
  * overall looks okay: full length (151 aa), 87% match, most differences are +
* AAC6-Ia
  * all hits (7) are specific (aminoglycoside 6'-N-acetyltransferase)
* AAC6-Ib  
  * one hit (C17)
  * putative aminoglycoside 6'-N-acetyltransferase
* AAC6-II
  * one hit (C17)
  * putative aminoglycoside 6'-N-acetyltransferase 
  * matches AAC6-Ib __databases/ models for -Ib and -II are not unique__
* CAT
  * 1 hit (Chloramphenicol acetyltransferase) 
* ClassA  
  * 6 hypothetical proteins (4.7%)
  * 126 total
  * hypotheticals are good matches and hit to ClassA elsewhere
* ClassB
  * 12 hypothetical proteins (4.8%)
  * 246 total
  * hypotheticals are good matches and hit to ClassB elsewhere
* ClassC  
  * 9 hypothetical proteins (27%)
  * 2 penicillin-binding protein (OK since its still B-lactam resistance)
  * 33 total
  * hypotheticals hit to ClassC elsewhere 
* intI
  * 10 tyrosine recombinases (OK, hits to integrase elsewhere)
  * 2 hypothetical proteins (OK, hits to integrase elsewhere)
  * 269
* tetA
  * 1 MFS transporter (OK, different name, same function)
  * 1 hypothetical protein
  * 1 PREDICTED: hippocampus abundant transcript 1 protein-like (second hit is tet efflux)
  * 1 major facilitator transporter
  * all have second (close) hits to tetA
* tetW  
  * 1 hypo (other hits tet)
  * other hits are tet resistance
* tetX
  * 3 hypothetical (OK, other thits tetX)
  * 1 FAD-binding monooxygenase (OK function of tetX)
* vanA  
  * 7 hypothetical proteins (17%) 
  * 41 total
  * other top hyp hits are vanA
* vanB
  * __databases/ models for vanA and vanB are not unique__
* vanH  
  * 2 hypothetical proteins (2.6%)
  * all other hits describe vanH 
  * 75 total
* vanX
  * 9 hypothetical proteins (8.8%)
  * many peptidase M15 (OK, other hits go to vanX)
  * 102 total
* vanZ
  * all top hits are vanZ
              
