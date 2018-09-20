# NGS Quality Control Tutorial: Understanding QC
This tutorial is intended to understand basic NGS statistics (mainly obtained with FastQC), and some of the steps required to  fix or ameliorate some of the issues. Most of the information of this tutorial have been partially taken from the FastQC documentation available [here](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/).

### Objectives / learning outcomes:

At the end of this tutorial you shoud be able to:

1. Understand the QC statistics used in FASTQC
2. Identify problems in NGS datasets
3. Remove adapters and primers 
4. Make simple quality trimming for reads

This tutorial assumes that you have a basic knowledge in bash, and that you have an account and know how to connect to the Compute Canada clusters. If you dont, I reccomend you go over the [BASH tutorial](https://github.com/jshleap/CristescuLab_misc/blob/master/Tutorials/Bash/Bash_Tutorial.ipynb) and you read the [Compute Canada documentation](https://docs.computecanada.ca/wiki/Compute_Canada_Documentation).

### Outline of the tutorial
1. [To start: A couple of things we need before we start the tutorial](#to-start)
2. [Getting the statistics with fastqc](#getting-the-statistics-with-fastqc)
3. [Basic Statistics](#basic-statistics)
4. [Per Base Sequence Quality](#per-base-sequence-quality)
5. [Per tile sequence quality](#per-tile-sequence-quality)
6. [Per sequence quality scores](#per-sequence-quality-scores)
7. [Per base sequence content](#per-base-sequence-content)
8. [Per sequence GC content](#per-sequence-gc-content)
9. [Per base N content](#per-base-n-content)
10. [Sequence Length Distribution](#sequence-length-distribution)
11. [Sequence Duplication Levels](#sequence-duplication-levels)
12. [Overrepresented sequences](#overrepresented-sequences)
13. [Adapter Content](#adapter-content)
14. [Running FastQC in the paired file](#running-fastqc-in-the-paired-file)

## To start

To start, let's download [this file](https://github.com/jshleap/CristescuLab_misc/raw/master/Tutorials/NGS_QC/files/file1_R1.fastq.gz) to your account in Compute Canada. Also, load the following modules fastqc:

```
module load fastqc/0.11.5
module load trimmomatic/0.36
```
 The file you downloaded is a real dataset from eDNA water samples. It is amplicon sequencing of a fragment of the 12S gene using Illumina's Nextera Libraries in paired end sequencing mode. The PCR amplification should have an average length of 163-185, however, is highly variable due to the multi species composition of the sample.
 
## Getting the statistics with fastqc

The statitics of any fastq file is easily obtained by the fastqc program. This program inludes a set of statistical test and modules to test for quality. From their README file:

> FastQC is an application which takes a FastQ file and runs a series
of tests on it to generate a comprehensive QC report.  This will
tell you if there is anything unusual about your sequence.  Each
test is flagged as a pass, warning or fail depending on how far it
departs from what you'd expect from a normal large dataset with no
significant biases.  It's important to stress that warnings or even
failures do not necessarily mean that there is a problem with your
data, only that it is unusual.  It is possible that the biological
nature of your sample means that you would expect this particular
bias in your results.

So let's run it!:

`fastqc file1_R1.fastq.gz`

This must have created an html file as well as a zipped folder. Use `rsync`, `scp`, [FileZilla](https://filezilla-project.org/), or your favorite file transfer protocol to get the html to your own computer, and open it in a browser.

## Basic Statistics

This table will give you basic information about your reads:
1. **Filename**: The name of the file being analyzed
2. **File type**: The type of information the file contains
3. **Encoding**: How are the quality scores encoded
4. **Total Sequences**: umber of reads in your file
5. **Sequences flagged as poor quality**: Number of sequences with very low quality thoughout
6. **Sequence length**: Average sequence length
7. **%GC**: Percentage of GC content

For our file we get:

![Basic stats for File1](https://github.com/jshleap/CristescuLab_misc/raw/master/Tutorials/NGS_QC/images/Basic_stats.png)

### Encoding
Encoding is the way the quality of the bases are written. There are many encodings, but the most popular are Sanger, Solexa, Ilumina 1.3+, Illumina 1.5+, and illumina 1.8+.  In summary, is a character that represents the confidence you have in a given base call.
![Phred Score encodings descriptions, from https://en.wikipedia.org/wiki/FASTQ_format#Encoding](https://github.com/jshleap/CristescuLab_misc/raw/master/Tutorials/NGS_QC/images/fastq_phread-base.png)
For more information check https://en.wikipedia.org/wiki/FASTQ_format#Encoding

But what does a quality score means? It s related to the probability of an error:
|Phred Quality Score |Probability of incorrect base call|Base call accuracy|
|--- |--- |--- |
|10|1 in 10|90%|
|20|1 in 100|99%|
|30|1 in 1000|99.9%|
|40|1 in 10,000|99.99%|
|50|1 in 100,000|99.999%|
|60|1 in 1,000,000|99.9999%|

As a rule of thumb a Phred score above 20 (99% chances to be right) is considered acceptable and above 30 (99.9% chances to be right) as good.

I am not going to enter the rest of the basic statistics since they are self-explanatory.

## Per Base Sequence Quality

It's name is self explanatory. This module evaluates the quality at each base for all reads. FastQC gives you a box plot of the qualities, representing the inter-quartile range (25-75%) (yellow box), the extremes 10 and 90th percentiles are represented by the whiskers, the median value by a red line, and the mean quality by the blue line.
![per-base quality of file 1](https://github.com/jshleap/CristescuLab_misc/raw/master/Tutorials/NGS_QC/files/file1_R1_fastqc/Images/per_base_quality.png)

From the documentation of this module:

> #### Warning 
> A warning will be issued if the lower quartile for any base is less than 10, or if the median for any base is less than 25.
> #### Failure
> This module will raise a failure if the lower quartile for any base is less than 5 or if the median for any base is less than 20.

#### *Look at the figure above. What do you think is happening at the end? why?*

## Per tile sequence quality
This a feature that is exclusive to Illumina technologies. Their flow cells typically have 8 lanes,with 2 columns and 50 tiles:

![Flow cell pattern](https://github.com/jshleap/CristescuLab_misc/raw/master/Tutorials/NGS_QC/images/illumina_flowcell.png)

Courtesy of http://zjuwhw.github.io/2016/08/13/Illumina_sequencer.html

When systematic error occur in a tile, it can indicate sequencing error such as bubbles, smudges, or dirt. When the errors occur very sparsely and not too widespread, is often OK to overlook this error. When a full lane has a problem, oftentimes is a sequencing error and this cannot be fixed with bioinformatics. The problem can occur as as well when the flowcell is overloaded.
In our case we have:
![Quality per tile](https://github.com/jshleap/CristescuLab_misc/raw/master/Tutorials/NGS_QC/files/file1_R1_fastqc/Images/per_tile_quality.png)

Not the best quality, but there is no systematic bias... we might be able to fix this with some quality trimming.

From FastQC documentation:
> #### Warning
> This module will issue a warning if any tile shows a mean Phred score more than 2 less than the mean for that base across all tiles.
> #### Failure
> This module will issue a warning if any tile shows a mean Phred score more than 5 less than the mean for that base across all tiles.

## Per sequence quality scores

This module allows you to explore if a significant portion of your reads are of poor quality. Often times warnings occur when your sequence is shorter than your read length, and therefore the end of reads (or the end of the flowcell) is of poor quality.

From FastQC documentation:
>#### Warning
>A warning is raised if the most frequently observed mean quality is below 27 - this equates to a 0.2% error rate.
>#### Failure
>An error is raised if the most frequently observed mean quality is below 20 - this equates to a 1% error rate.


This is the case for our File1:
![Per sequence quality scores of File1](https://github.com/jshleap/CristescuLab_misc/raw/master/Tutorials/NGS_QC/files/file1_R1_fastqc/Images/per_sequence_quality.png)

#### *Can you explain the figure above?*


## Per base sequence content
This module shows the proportion of bases in each position. In an unbiased library, the proportion of A, T, C, G, should run parallel to each other. If there is a bias, this could imply that the primers or adaptors were not remove, and therefore there would be a strong bias towards a certain composition. It could also mean that you have an over-fragmented library, creating over-represented k-mers, or a dataset that has been trimmed too aggressively. In amplicon sequencing, there tends to be biases in the composition of the given amplicon, especially when dealing with mitochondrial DNA.

From FastQC documentation:
>#### Warning
>This module issues a warning if the difference between A and T, or G and C is greater than 10% in any position.
>#### Failure
>This module will fail if the difference between A and T, or G and C is greater than 20% in any position.

 Let's take a look at file1:

![Per sequence base content for file1](https://github.com/jshleap/CristescuLab_misc/raw/master/Tutorials/NGS_QC/files/file1_R1_fastqc/Images/per_base_sequence_content.png)

#### What can you tell about this file?

## Per sequence GC content
This module intends to show the proportion of GC content in the reads. The blue line represents a theoretical distribution (Normal) of your observed data. Deviations from this theoretical distribution often implies contamination of some kind (adapter/primer dimers, multiple species in the run). FastQC assumes that you are analyzing a  single genome, and therefore will issue a warning in multispecies libraries.
From FastQC documentation:
>#### Warning
>A warning is raised if the sum of the deviations from the normal distribution represents more than 15% of the reads.
>#### Failure
>This module will indicate a failure if the sum of the deviations from the normal distribution represents more than 30% of the reads.
>#### Common reasons for warnings
>Warnings in this module usually indicate a problem with the library. Sharp peaks on an otherwise smooth distribution are normally the result of a specific contaminant (adapter dimers for example), which may well be picked up by the overrepresented sequences module. Broader peaks may represent contamination with a different species.

 Let's take a look at out file1:

![Per sequence GC contentent](https://github.com/jshleap/CristescuLab_misc/raw/master/Tutorials/NGS_QC/files/file1_R1_fastqc/Images/per_sequence_gc_content.png)

#### How would you explain the two modes (double peak)?

## Per base N content
Some sequencer technologies would produce an N when it cannot define which of the four bases it has confidence on based on the phenogram. Illumina does not produce this, and therefore the plot should be flat and the module should always pass.

From FastQC documentation:
>#### Warning
>This module raises a warning if any position shows an N content of >5%.
>#### Failure
>This module will raise an error if any position shows an N content of >20%.

Failure or warning in this module suggest that the sequencing should probably be repeated since a significant portion of your reads have no information in them.

Since our toy file is Illumina, it shows a flat line in the bottom of the figure:

![per base N content](https://github.com/jshleap/CristescuLab_misc/raw/master/Tutorials/NGS_QC/files/file1_R1_fastqc/Images/per_base_n_content.png)

## Sequence Length Distribution
It self explanatory title describes well this module. It plots the distribution of sequence length for your reads. Illumina produces the same length throughout all the lanes, however, other sequencing platforms produce a distribution of them. This module can be safely ignore if you know that you are expecting a population of lengths in your reads. If you are using illumina and this module fails or gives you a warning, you should talk to the provider of the sequencing.

Our dummy file (since is illumina) behaves as expected:

![Length distribution of file1](https://github.com/jshleap/CristescuLab_misc/raw/master/Tutorials/NGS_QC/files/file1_R1_fastqc/Images/sequence_length_distribution.png)  

## Sequence Duplication Levels
This module allows you to see the level of duplication of your library. Ideally, the blue (total sequences) and the red (deduplicated sequences) should match. This would mean that you had a diverse library, and that each sequence has been sequenced in the proper depth. However, this assumes that you are working with a genome of single species, and therefore the warnings and failures of this module should only be worrysome then, since it will show a bias (i.e. PCR artefacts, resequencing parts of genome). In enriched libraries, you would expect some level of duplication, especially when this module only takes the first 50 bases and the first 100K sequences to run the tests. In amplicon sequencing, we expect some degree of duplication, and we should not be too agressive in cleaning this up.

From FastQC docs:

>#### Warning
>This module will issue a warning if non-unique sequences make up more than 20% of the total.
>#### FailureThis module will issue a error if non-unique sequences make up more than 50% of the total.
In our file, we get:

![Dupication levels](https://github.com/jshleap/CristescuLab_misc/raw/master/Tutorials/NGS_QC/files/file1_R1_fastqc/Images/duplication_levels.png)

#### Explain the figure above

## Overrepresented sequences
This cool module shows you sequences that are present in over 0.1% of your total reads. The coolest thing about it is that it will run a search for common contaminants and report them. In a single species, diverse, uncontaminated library, you should expect not to have any overrepresented library.

Check your copy of the overrerpesented sequences in the html file. Here is a screenshot of the first par:
![Overrepresented sequences for file1](https://github.com/jshleap/CristescuLab_misc/raw/master/Tutorials/NGS_QC/images/overrepresented.png)

#### What can you tell me about it? How would you check if is OK or not? Do it!!!Overrepresented sequences

## Adapter Content
Another self-explanatory module. Here, the most commonly used adapters are screened for. They are mostly illumina adapters (Universal, Small 3' RNA, Small 5' RNA, Nextera) and SOLiD small RNA adapter. From the docs:
>Any library where a reasonable proportion of the insert sizes are shorter than the read length will trigger this module. This doesn't indicate a problem as such - just that the sequences will need to be adapter trimmed before proceeding with any downstream analysis.
 
 In our file:
 ![enter image description here](https://github.com/jshleap/CristescuLab_misc/raw/master/Tutorials/NGS_QC/files/file1_R1_fastqc/Images/adapter_content.png)

## Running FastQC in the paired file
Now, let's [download the pair](https://github.com/jshleap/CristescuLab_misc/raw/master/Tutorials/NGS_QC/files/file1_R2.fastq.gz) or R2 file. Run FastQC and tell me what do you think about this second file. Compare it to the first one. What do you see?

# Resolving some of the Issues
There are many programs to do QC, and many specific tools for each one. For now we are going to focus on a few popular programs.

## Trimmomatic
This program does adaptve quality trimming, head and tail crop, and adaptor removal. You can check the documentation and download the program [here](http://www.usadellab.org/cms/index.php?page=trimmomatic). One of the advantages of trimmomatic is that it allows you to work with pair end sequences, reatining only matching pairs. Other advantage  is that it allows partial and overlaping matches for the seaching of adapters. Before we run the program, let's check at some of the options. Here I am going to focus in pair-end reads:
 
 ### Efficiency and format flags
 This flags go before the invocation of the output/input files:
 1. -threads: this flag modifies the number of cpu threads that trimmomatic should use in the computations. A typical laptop computer have about 2 cores which should amount to 4 available threads.
 2.  [-phred33 | -phred64]: this flags tells trimmomatic the encoding of the file (see [above](#encoding))
 
 ### Change encoding option
 If you want to read your file in one encoding and output it in a different one, this options are the ones you need to use:
- TOPHRED33: Convert quality scores to Phred-33
- TOPHRED64: Convert quality scores to Phred-64
This options (and all the following on CAPS) must go after the input/output files.

### Cropping
Trimmomatic has several options that can be use simultaneously or not:
-   LEADING: Cut bases off the start of a read, if below a threshold quality
-   TRAILING: Cut bases off the end of a read, if below a threshold quality
-   CROP: Cut the read to a specified length
-   HEADCROP: Cut the specified number of bases from the start of the read

LEADING and TRAILING are adaptive cropping. That means that they will cut your read's head and/or tail if they fail the specified quality. This differs from CROP and HEADCROP, which would cut at an specified length or specified number of bases respectively. For the latter two, the program will perform the cropping for all reads

### Adaptive length filtering
Trimmomatic has the option MINLEN which will drop reads that fall under the specified length:
-   MINLEN: Drop the read if it is below a specified length

### Adaptive quality trimming
The SLIDINGWINDOW option allows you to trimm reads based on their average quality in a window:

-   SLIDINGWINDOW: Perform a sliding window trimming, cutting once the average quality within the window falls below a threshold.

It takes two values like `SLIDINGWINDOW:4:15` which means  "Scan the read with a 4-base wide sliding window, cutting when the average quality per base drops below 15"

### Adapter trimming
Finally, trimmomatic will take a file with the sequences  of your adapters and will trimm them out. It follows the following call: `ILLUMINACLIP:<fastaWithAdaptersEtc>:<seed mismatches>:<palindrome clip
threshold>:<simple clip threshold>`. From their docs:
> - fastaWithAdaptersEtc: specifies the path to a fasta file containing all the adapters,
PCR sequences etc. The naming of the various sequences within this file determines
how they are used. See the section below or use one of the provided adapter files
> - seedMismatches: specifies the maximum mismatch count which will still allow a full
match to be performed.
>- palindromeClipThreshold: specifies how accurate the match between the two 'adapter
ligated' reads must be for PE palindrome read alignment.
>- simpleClipThreshold: specifies how accurate the match between any adapter etc.
sequence must be against a read.

![Adapter trimming](https://github.com/jshleap/CristescuLab_misc/raw/master/Tutorials/NGS_QC/images/trimmomatic_adapter.png)

As can be seen in the figure, there are 4 possible scenarios that trimmomatic cover:
A. Technical sequence is completely covered by the read and therefore a simple alignment will identify it.
B. Only a partial match between the technical sequence and the read, and therefor a short alignment is needed.
C and D.  Both pairs are tested at once, hence allowing for "is thus much more reliable than the short alignment in B, and allows adapter read-though to be detected even when only one base of the adapter has been sequenced."

### Running trimmomatic
`java -jar <path to trimmomatic.jar> PE [-threads <threads] [-phred33 | -phred64] [-trimlog <logFile>] <input 1> <input 2> <paired output 1> <unpaired output 1> <paired output 2> <unpaired output 2> <OPTIONS>`
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTg2Nzk2NDcxMywtOTQ5ODk3NjkxLDE1MD
Q3NTUzMDEsODQzNjU4MTUsLTU3NjgyNzY4Miw1NzMzNDI0NTks
NzY5NjQ1NDg0LC0xMjYxMTIzOTcwLC04ODI0NTMwMDUsLTI1NT
Q0NDAwMiwxMDE2OTMxNTgyLC0xNjI2NzcyOTMwLDExNTYyOTI4
NTYsLTEzMjIxMDMyOTUsLTc5NzYwNzM0LDE1MjM1MDQ0ODUsMj
c4NTgyNzg3LDEyOTU0OTAxMjQsMjA5OTk4MjQ4NSwxMzY3NTgy
MjYyXX0=
-->