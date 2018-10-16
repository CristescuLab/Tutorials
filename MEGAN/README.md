# Introduction to MEGAN methods

This tutorial (more like a conversation and notes), will try to introduce to some aspects of the Software [MEGAN](http://www-ab.informatik.uni-tuebingen.de/software/megan6/). I will assume that you are more or less familiar with metagenomics pipelines, and will focus more on how MEGAN does things. 
The papers and manual that you can check and where I got graphics and information are:
-   [Huson et al.,MEGAN Community Edition - Interactive exploration and analysis of large-scale microbiome sequencing data,  PLoS Computational Biology, 2016, 12 (6): e1004957](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1004957)
-   [Buchfink et al, Fast and sensitive protein alignment using DIAMOND, Nature Methods, 2015, 12:59-60.](http://www.nature.com/nmeth/journal/v12/n1/full/nmeth.3176.html)
-   [Huson et al, Integrative analysis of environmental sequences using MEGAN4, Genome Res, 2011, 21:1552-1560.](http://genome.cshlp.org/content/21/9/1552.long)
-   [Huson et al, MEGAN analysis of metagenomic data, Genome Res, 2007, 17(3):377-86.](http://genome.cshlp.org/content/17/3/377.long)
- [User Manual for MEGAN V6.12.3](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=2ahUKEwjinvOD6cTdAhXHxYMKHUy-AekQFjAAegQIBRAC&url=http%3A%2F%2Fab.inf.uni-tuebingen.de%2Fdata%2Fsoftware%2Fmegan6%2Fdownload%2Fmanual.pdf&usg=AOvVaw0RSRcHsae9qWupV1_G_ln6)

There are also useful tutorials:
- [Megan Tutorial 2018.key - Algorithms in Bioinformatics](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=2&ved=2ahUKEwjcs5S86cTdAhVo9IMKHRfPA-cQFjABegQICRAC&url=http%3A%2F%2Fab.inf.uni-tuebingen.de%2Fdata%2Fsoftware%2Fmegan6%2Fdownload%2FMeganTutorialApril2018.pdf&usg=AOvVaw3hBaeMzeFdB-brR50gAXFq)
- [Taxonomic annotation](https://metagenomics-workshop.readthedocs.io/en/latest/annotation/taxonomic_annotation.html)
- [Metagenomics Workshop SciLifeLab Documentation - Read the Docs](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=14&ved=2ahUKEwiGksOl6sTdAhWK3oMKHQfOCoYQFjANegQICRAC&url=https%3A%2F%2Freadthedocs.org%2Fprojects%2Fmetagenomics-workshop%2Fdownloads%2Fpdf%2Flatest%2F&usg=AOvVaw3x4abtP7EXWdmJloj9qZ14)

## Objectives / learning outcomes:

At the end of this tutorial you shoud be able to:

 1. Better understand the MEGAN pipeline
 2. Underderstand and make use of DIAMOND
 3. Know the file formats used in MEGAN
 4. Understand the taxonomic assignment by LCA (last common ancestor)
 5. Use command line script MEGANIZER to process data
 6. Get a gist on the Long Read LCA algorithm: mainly longer contigs
 7. Have an idea of the visualization of your data and general analyses
 
 
## Outline of the tutorial
1. [The MEGAN pipeline: a general overview](#the-megan-pipeline)
2. [DIAMOND: a lighting fast blast](#diamond-a-lighting-fast-blast)
3. [Meganizer: from .daa to MEGAN](#meganizer-from-.daa-to-megan)
4. [LCA taxonomic binning](#lca-taxonomic-binning)
5. [MEGANServer: Connecting a server and the app](#meganserver-connecting-a-server-and-the-app)
6. [Visualization with MEGAN](#visualization-with-megan)

## The MEGAN pipeline

![MEGAN pipeline](https://journals.plos.org/ploscompbiol/article/figure/image?size=large&id=10.1371/journal.pcbi.1004957.g001)

Overall, MEGAN pipeline can be divided in 4 steps:
1. The input files (already QCed) are analyzed with Diamond (we will get to this in a sec)
2. Diamond (or blast) outputs are converted into MEGAN DAA files
3. Storage in server
4. Interactive inteaction

Basically, you align your reads to a database, bin your reads with annotations, and finally visualize the results. Given that most of your are comfortable with the graphic interface of MEGAN, I will focus on the first three steps.

## DIAMOND: a lighting fast blast 
DIAMOND stands for **D**ouble **I**ndex **A**lign**M**ent **O**f **N**ext-generation sequencing **D**ata, and you can get it [here](http://ab.inf.uni-tuebingen.de/software/diamond), and if you haven't install it now! They report that DIAMOND is ~20,000 times faster than BLASTX. DIAMOND, as MEGAN, are designed for metagenomics, particularly to explore species/OTUs profile, and therefore is usual to align reads to protein databases (hence usage blastx). 

To understand the underlying algorithm of DIAMOND, we need to compare with the traditional blast. Both used the seed and extend strategy (images from [Alarfaj et al.](https://www.doi.org/10.4172/jcsb.1000260)):
##### SEED
![enter image description here](https://raw.githubusercontent.com/jshleap/CristescuLab_misc/master/Tutorials/Blast/img/Seeding-step.png)
##### EXTEND
![enter image description here](https://raw.githubusercontent.com/jshleap/CristescuLab_misc/master/Tutorials/Blast/img/extension_step.png)

However, DIAMOND modifies the seeding in two ways:
1. Reduced alphabet: Their empirical studies allowed them to "re-write" the protein alphabet. This rewrite is done by grouping aminoacids into 11 categories: [KREDQN] [C] [G] [H] [ILV] [M] [F] [Y] [W] [P] [STA].
2. Spaced seeds: Allowing for different "shapes" of seeds and spaces within each seed.
![enter image description here](https://media.nature.com/lw926/nature-assets/nmeth/journal/v12/n1/images_supplementary/nmeth.3176-SF1.jpg)
3. Seed index: In short (lots of CS jargon in this), the query is pre-processed and check for common seeds. Each grouped seed is associated with the query names that contains it so that the access during the alignments is faster. 
4. Double indexing: Again, lots of CS jargon, but putting symply seeds from subject and query are both index and sorted, making the matching very fast.
![enter image description here](http://pangenome.tuebingen.mpg.de/images/diamond.png)

More details of the DIAMOND algorithm can be found in [this very nice tutorial](http://ab.inf.uni-tuebingen.de/teaching/ws14/bioinf/13-BeyondBlast.pdf), and I encourage everybody to read it and bring doubts to other tutorials. From that tutorial:
>In summary, here is an outline of the DIAMOND algorithm.
>1. Input the list of query sequences $Q$.
>2. Translate all queries, extract all spaced seeds and their locations, using a reduced alphabet.
>3. Sort them. Call this $S(Q)$.
>4. Input the list of reference sequences $R$.
>5. Extract all spaced seeds and their locations, using a reduced alphabet. Sort them. Call this $S(R)$.
>5. Traverse $S(Q)$. and $S(R)$ simultaneously. For each seed S that occurs both in $S(Q)$ and $S(R)$, consider all pairs of its locations x and y in $Q$ and $R$, respectively: check whether there is a left-most seed match. If this is the case, then attempt to extend the seed match to a significant banded alignment.

Now, let's compare how DIAMOND does against BLAST. To do so, we need to format the databases both for blastx, and for DIAMOND. I assume you are versed in blast command line tools, and if not, I encourage you to read the [tutorial](https://github.com/jshleap/CristescuLab_misc/tree/master/Tutorials/Blast) we did a couple of sessions ago. If you are part of the Cristescu Lab, the pre-formated diamond database for nucleotides can be found in our database folder under NCBI. If you are not, please download [this file](ftp://ftp.ncbi.nlm.nih.gov/blast/db/FASTA/nr.gz) (unfortunately it takes a while and 45Gb). Wether you are part of the CristescuLab or not, format a database (for the Cristescu lab, use a mock fasta file) with the `diamond makeblastdb` command:
```
cd PATH/TO/FASTA
diamond makedb --in <FASTA_FILE> -d <NAME_DB>
```
Replace PATH/TO/FASTA, FASTA_FILE, and NAME_DB with your actual values. In the CristescuLab `~/projects/def-mcristes/Databases/NCBI`, I have pre-formatted the full nt database by downloading the fasta file of the entire database (and hence the 45Gb) and executed:
`zcat nr.gz| diamond makedb -d diamond_nr`

Once we have done this, we are ready for the test with [this mock fasta file](https://raw.githubusercontent.com/jshleap/CristescuLab_misc/master/Tutorials/MEGAN/files/sample.fa).

Let's now run a blastx with both BLAST and DIAMOND algorithms (**_remember to load them or download them_**):
`time blastx -db PATH/TO/DATABASE/nr -query /PATH/TO/QUERY/sample.fa -out <OUTFN> -num_threads <CPUS>`
**_Just kidding don't run it... it will take several hours. If you want you can download the results [here](https://raw.githubusercontent.com/jshleap/CristescuLab_misc/master/Tutorials/MEGAN/files/submitted.blast)_**
This one you can run though! **Make sure to use a salloc with at least 32 cpus**
`time diamond blastx -d PATH/TO/DATABASE/diamond_nr -q /PATH/TO/QUERY/sample.fa -f 100 --salltitles -o <OUTFN>.daa`
Or you can just get the result [here](https://raw.githubusercontent.com/jshleap/CristescuLab_misc/master/Tutorials/MEGAN/files/diamond.daa).
Now we have a .daa file (-f is the format, 100 is the binary output format) which basically is the DIAMOND output. 

##### But not everything shines!
In my experience the accuracy drops significantly even with the more sensitive approach. Not great for accurate functional annotation when false positives want to be controlled.

## Meganizer: from .daa to MEGAN

After we have a DIAMOND binary output format (MEGAN call it .daa). This file format is DIAMOND proprietary format, but MEGAN supports it. Before we dig into Meganizer, lets take a peak of this file with DIAMOND:
`diamond view --daa OUTFN.daa | head`

The MEGAN user interface uses a different compressed file format called RMA. This is essentially a compressed and comprehensive file that summarizes annotation and classifications to be visualized with the MEGAN GUI. This step takes the information from the alignment to a database (either DIAMOND or BLAST), the taxonomic information, and functional information, and append them in a single file in a relational manner that MEGAN-CE can understand. We can do this either with `daa-meganizer` or with `daa2rma`, but we need mapping files between the reference database and the functional annotations. For this example, lets focus in the KEGG annotation, and the mapping file can be downloaded [here](http://ab.inf.uni-tuebingen.de/data/software/megan6/download/acc2kegg-Dec2017X1-ue.abin.zip), and the taxonomic mapping can be found [here](http://ab.inf.uni-tuebingen.de/data/software/megan6/download/prot_acc2tax-June2018X1.abin.zip). These files need to be unzipped. Now let's check some of the options first:
`PATH/TO/daa2rma -h`

Therefore if we want to include the KEGG annotations we will do:
`PATH/TO/daa2rma -i OUTFN.daa -o reads.rma -a2t prot_acc2tax-June2018X1.abin -a2kegg acc2kegg-Dec2017X1-ue.abin.zip -fun KEGG`

### LCA taxonomic binning
The LCA algorithm is explained in [this paper](https://genome.cshlp.org/content/17/3/377.full).
The LCA algorithm has three types: naive, weighted, longReads. The naive algorithm is nice and simple:

![enter image description here](https://genome.cshlp.org/content/17/3/377/F2.large.jpg)

Basically,  each read is assigned to the lowest common ancestor (LCA) of the set of taxa that had a significant hit in the comparison, and these are binned on that LCA. The taxonomic assignments and the tree above are based exclusively in the NCBI taxonomy (you can add other taxonomies like SILVA). The naive approach bases its assignment solely in the presence/absense of hits between reads.

The weighted LCA has the similar simplicity in mind. However, not only the presence/absense of hits between reads is taken into account, but the number of uniquely aligned hits (kind of similar to OTU abundances).

The LongRead LCA is devised for longer reads and/or contigs. This is the newer development of MEGAN, explained in [this paper](https://biologydirect.biomedcentral.com/articles/10.1186/s13062-018-0208-7).
<p align="center">
<img src="https://media.springernature.com/full/springer-static/image/art:10.1186/s13062-018-0208-7/MediaObjects/13062_2018_208_Fig1_HTML.gif" width="680" height="600">
</p>
The read is split into intervals (regions) based on the alignment with regions in the reference database. Put in the authors' words:
> a new interval starts wherever some alignment begins or ends.

The significance of the alignment is defined  based on the best bitscore: "an alignment is significant if lies within the 10% of the best bit score". This is a tunable parameter.


## MEGANServer: Connecting a Linux server and the app
WHY?? Because the sizes and complexities of metagenomics datasets can be bigger than desktop computers, and to make easier the sharing of data.

> With MeganServer one outsources the storage of metagenomic datasets to a different computer and accesses their content via MEGAN.

To set up a server you have to download the [standalone](http://www-ab.informatik.uni-tuebingen.de/data/software/meganserver/download/MeganServer-standalone.1.0.1.zip) version of meganserver, and setup the start script. MeganServer can be then access through the MEGAN GUI.

## Visualization with MEGAN
MEGAN have many utilities to visualize your data, here just a few examples (focusing more in non-functional exploration):

### Basic visualization
<p align="center">
<img src="https://github.com/CristescuLab/Tutorials/raw/master/MEGAN/files/basic_tree.jpg" width="680" height="600">
</p>

### Comparative visualization
In MEGAN one can upload results of multiple datasets and have a comparative view the relative abundance of reads in each sample or group:

<p align="center">
<img src="https://www.researchgate.net/profile/Suparna_Mitra/publication/221686029/figure/fig4/AS:305399644344323@1449824357872/Comparative-visualization-of-eight-marine-datasets-8-displaying-the-bacterial-part-of.png" width="680" height="600">
</p>

<p align="center">
<img src="https://media.springernature.com/lw785/springer-static/image/art%3A10.1186%2F1471-2105-10-S1-S12/MediaObjects/12859_2009_Article_3195_Fig4_HTML.jpg" width="680" height="600">
</p>


<p align="center">
<img src="https://ab.inf.uni-tuebingen.de/software/megan4/images/MEGAN-taxonomy-Goodall-neighbor-net.png/image_preview" width="680" height="600">
</p>

## Practice time!



## Time Permitting: BASTA
![enter image description here](https://i0.wp.com/www.palmbeachcountycta.org/wp-content/uploads/2017/10/website-construction-graphic-4.jpg?w=1024)
