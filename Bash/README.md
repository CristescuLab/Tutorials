
# A Very very Basic BASH tutorial

This very basic tutorial is to introcuce a complete novice in the use of bash. The idea is that the person will be able to move around the Compute canada Clusters and be more or less comfortable with it. It is by no means comprehensive nor is going to make you a bash ninja, but it will allow you to deal with clusters, and some minor manipulations.


## Don't be afraid of the terminal

The terminal is your friend, and you'll see why. The terminal is the way we talk directly to the computer without windows and cliking. It looks something like this:

<img src="files/images/Terminal.png" width="500px">

If you work in windows you will need to install an ssh-enabled terminal like [mobaxterm](https://mobaxterm.mobatek.net/) or [putty](https://www.putty.org/). This tutorial will asume you already have installed and read the instructions of your terminal.

For MAC and linux users, just search for terminal, and voila!

##### TRY TO AVOID FILENAMES WITH SPACES IN THE TERMINAL...IS ANNOYING

## SSH: Connecting to the cluster
Since most of this tutorial is meant to help you use the Compute Canada clusters, I'll start with the connection. SSH stands for Secure SHell (shell is the terminal), and you just have to type `ssh username@clustername.computecanada.ca`, where `username` is your own username in Compute Canada and `clustername` is the name of the cluster. I believe most of use use Graham or Cedar. In this tutorial I'll assume we will be working in graham. 


```python
ssh jshleap@graham.computecanada.ca
```

    Pseudo-terminal will not be allocated because stdin is not a terminal.
    ssh: Could not resolve hostname graham.computecanada: Name or service not known


Once you have typed your password, you are in the cluster!!!


## Moving around the cluster
If you only have a single Role (seehttps://ccdb.computecanada.ca/me/faq#what_is_role for the definition) and if you have never worked in this account before you should have two folders inside your account. To see the folders or the content of anything, we will need to ask the terminal to list the contents:


```python
ls
```

    Bash Tutorial.ipynb  [0m[01;34mimages[0m/  README.md  Slurm_vs_PBS.ipynb


To move to a directory you need to use the change directory command `cd`


```python
cd images

```

    /home/jshleap/my_gits/CristescuLab_misc/images



```python
ls

```

    [0m[01;35mTerminal.png[0m


Compute Canada clusters work with Unix operating systems (Not windows). This means that there are a few different things. The way the terminal knows where you want to go i thorugh the path. Paths are just a road map to the location you want to be in.  For example, lets imagine that we would like to go to a subdirectory called tutorial from my hme directory, there are two ways to do this:


```python
cd /home/jshleap/tutorial
```

    [Errno 2] No such file or directory: '/home/tutorial'
    /home/jshleap/my_gits/CristescuLab_misc/images



```python
cd ~/tutorial
```

    [Errno 2] No such file or directory: '/home/jshleap/tutorial'
    /home/jshleap/my_gits/CristescuLab_misc/images


The tilde (~) is a code for your home directory. In this notebook it fails because in my desktop this folder does not exist. So let's create one!!!


```python
mkdir ~/tutorial # this command tell the computer to create a folder named tutorial in your home directory
```

now we can move to that folder:


```python
cd ~/tutorial
```

    /home/jshleap/tutorial


If you list the contents here with `ls` as before, you will notice that it is an empty folder


```python
ls
```

Now lets create an empty file using `touch`. This is not the most useful command, but allow you to know if you can write in the folder you are in, among other minial things.


```python
touch afile.txt 
```

Now if you type `ls` again, the afile.txt should appear. Lets try to edit this file. There are many tools, and depending on how you are sshing into the clusters they might change. I will focus in the native reader of unix **nano**:


```python
nano afile.txt
```

You should see something like this:
<img src="files/images/nano.png" width="500px">
You can write anything you want there. Lets try just typing "this is a test file". To close and save you have to press the ``ctrl + x``. It will ask you if you want to save what you just write, type `Y` and `enter`.
So you might be asking if you have to open nano everytime you need to read the file without modyfying it? The answer is no. There are a few commands you can use:
* cat if your file is small
* head if you want to see the first few lines of your file
* tail if you want to see the last few lines of your file
If you execute `cat afile.txt` it will show you what you just wrote. Cat will output the entire content of the file to screen. That is why is only wise to use it in small files or for certain manipulations (outside of the scope of this tutorial). The same way we can use head `head afile.txt` and tail `tail afile.txt`. In the litle single line file, all three commands should show exactly the same.

Now let's remove that file with the command `rm`:


```python
rm afile.txt
```

Be very careful with the usage of `rm`, since **once you have use it in a file you cannot recover the file**.

## Downloading from the web
Now, we need a more substantial file to play around with, so let's download one from the web directly into the cluster!!! For this we will use the command `wget`:


```python
wget https://raw.githubusercontent.com/jshleap/CristescuLab_misc/master/Tutorials/Bash/test_files/allponds.fasta
```

This will download a fasta file with some sequences. As you probably know, a fasta file (!!!NOT A FASTQ) is formated like:
```
>SEQNAME OR INFO FOR SEQ1
SEQUENCE
>SEQNAME OR INFO FOR SEQ1
SEQUENCE
```

## Exploring a file
Now let's explore this file with the `cat`, `head`, and `tail` commands:


```python
cat allponds.fasta
```


```python
head allponds.fasta
```


```python
head -n 5 allponds.fasta
```

The `-n` flag in head tells the command how many lines to output!! Play around with it! The `tail` command works very similar but instead of showing the first `n` lines, it shows the `n` last:


```python
tail -n 5 allponds.fasta # play around with it!
```

#### Explore the file page by page
Let's say that you'd like to see the file page by page instead of the whole thing at once and with more than a given number of lines. You can do this with the comands `less` and `more`.


```python
more allponds.fasta # to quit press Q,  to move return or space bar
```


```python
less allponds.fasta # to quit press Q to move use arrows or space bar
```

Let's imagine that you would like to see the total number of lines that this file has, you can use the word count command `wc`:


```python
wc allponds.fasta
```

     1000  1200 67931 ./test_files/allponds.fasta


This tells us that the file has 1000 lines, 1200 words and 67931 characters. If we want just the number of lines we can use `wc -l`:


```python
wc -l allponds.fasta
```

    1000 ./test_files/allponds.fasta


Great!! now we know that we have a 1000 lines file, but how many sequences do we have? Because allponds.fasta is a fasta (**not a fastq**), the sequences are indicated by the `>` symbol. This should be a unique symbol in fasta BUT NOT IN FASTQ. We will need to search occurencies of this symbol. Fortunately, bash have the very handy command `grep'.


```python
grep '>' allponds.fasta
```

    >M00833:667:000000000-BV9MW:1:2106:28755:11977 1:N:0:TAAGGCGA+GCGTAAGA;size=685;
    >M00833:667:000000000-BV9MW:1:2107:4035:17001 1:N:0:TAAGGCGA+GCGTAAGA;size=510;
    >M00833:667:000000000-BV9MW:1:2106:27458:12091 1:N:0:TAAGGCGA+GCGTAAGA;size=431;
    >M00833:667:000000000-BV9MW:1:2106:17802:12685 1:N:0:TAAGGCGA+GCGTAAGA;size=268;
    >M00833:667:000000000-BV9MW:1:2106:23093:11024 1:N:0:TAAGGCGA+GCGTACGA;size=217;
    >M00833:667:000000000-BV9MW:1:2107:17986:18261 1:N:0:TAAGGCGA+GCGTAAGA;size=200;
    >M00833:667:000000000-BV9MW:1:2107:24169:15780 1:N:0:TAAGGCGA+GCGTAAGA;size=198;
    >M00833:667:000000000-BV9MW:1:2106:7985:12895 1:N:0:TAAGGCGA+GCGTAAGA;size=163;
    >M00833:667:000000000-BV9MW:1:2109:9233:9403 1:N:0:TAAGGCGA+GCGTAAGA;size=137;
    >M00833:667:000000000-BV9MW:1:2107:19829:15122 1:N:0:TAAGGCGA+GCGTAAGA;size=99;


Well, this gave us all the lines where `>` is found in the file, which is useful, but not what we where looking for. We would like to count the lines to know the number of sequences we have. Fortunately, grep has the -c option:


```python
grep -c '>' allponds.fasta
```

    200


Perfect!! it tell us that there are 200!!! Can this fail? YES! if there are more `>` in the file in lines that are not the sequence. However, if you had a `>` is the sequence of a fasta file, it would be a wrongly formatted file. Why am I telling you this? beacuse dealing with fastq files is a different story. I will get back at this later in the tutorial, but keep it in mind.


### To recap:
1. To list all files and directories use `ls`
2. To move to a given directory use `cd`
3. To create a folder use `mkdir`
4. To remove a **file** use `rm`
5. To create an empty file use `touch`
6. To natively edit a file use `nano`
7. To explore a file page by page use `less` or `more`
8. To count the number of lines, words and characters in a file use `wc`
9. To count exclusively the number of lines in a file use `wc -l`
10. To find a pattern in a file use `grep`
11. To find a pattern in a file and count the occurences use `grep -c`

## Regular expressions, redirection, and pipes
OK, so far we can move from one directory to another, we can create directories and files, remove files, and find characters and words in a file. But there are a bunch of useful things that we can use these same commands that expands their usability. One of these things is the usage of regular expresions. According to [wikipedia](https://en.wikipedia.org/wiki/Regular_expression) is *"a sequence of characters that define a search pattern"*. With some special characters we can expand searches for example. One of the most used one is the wildcard `*`. This means that in that position it can be anything. Let's use it with the list `ls` command, but first lets create a bunch of files with a given pattern:


```python
#this is a loop. We are not covering this in this tutorial, but have a look if you are interested
for i in {1..10}; do touch prefix${i}.txt;done
```

The code above will create 10 files. list them using `ls`. Now with only `ls` you can see all files, including `allponds.fasta` and the previous files that we have created. Now, if we would like to list only the files that end in `.txt`, we can:


```python
ls *.txt
```

This command tells the computer to replace `*` for anything (ergo wildcard), and therefore list anything that ends in `.txt`. But if we have multiple `.txt` files and you would like to list a particular pattern, you can also use the wildcard:


```python
ls prefix*.txt
```

This just list all the files that start with `prefix`, and end with `.txt`, no matter what it is in the middle. This strategy can be used with most commands. To test, write something in each of the 10 files, and then lets do:


```python
cat prefix*.txt
```

It must print to screen a concatenation of all the files you wrote on. Now, what if you would like to merge all this files? well we can use the same strategy as before but we will **redirect** the output to a file, using the redirection command `>`:


```python
cat prefix*.txt > merged.txt
```

Check the contents of the new file! 
Another useful regex (a.k.a. regular expresion) that is useful is the `^` symbol. It means the begining of a line! As an example, let's download a fastq file and try to count the lines:


```python
wget https://raw.githubusercontent.com/jshleap/CristescuLab_misc/master/Tutorials/Bash/test_files/example.fastq
```

Explore the file with the tools you now know how to use!! Place special attention to the symbol `@`. In a fastq file, `@` refers both to the sequence ID, as well as a quality Score when the `@` is at the quality section. Because of this we cannot count the number of sequences by a plain `grep -c '@' example.fastq`. If we do, we will be counting also the quality line of the file. Try grepping the `@` without the `-c` flag:


```python
grep '@' example.fastq
```

Difficult to see everything? No problem! there is another bash trick, PIPES! You can feed one command with the output of other command. Let's try to get the fist 10 lines of the grep output we just had. We saw that head does that with a file, but if we want head to get the output of grep we use the pipe (`|`) command like:


```python
grep '@' example.fastq | head -n 10
```

It also works with `more` or `less` commands!:


```python
grep '@' example.fastq | more
```

Checking the file you would see that if we use `grep '@'` we will double count in many parts of the file. It would be better if we could grep only when `'@'` is at the begining of the line. To do that we go back to the regular expresion `^`. If we do:


```python
grep '^@' example.fastq
```

We can see that most of all of our output starts with `@` and most of it corresponds to the sequences. However some instances of our output will contain non-seqid lines, so counting will over estimate the number of sequences in the fastq file. To resolve this we need to find a pattern unique to the seqID. In this particular case, all sequences start with `M`. Let's explore this with `grep` and regex:


```python
grep '^@M' example.fastq | more
```

Now we only see sequences IDs. So how many sequences we have?


```python
grep -c '^@M' example.fastq
```

Now we have a more realistic count of our sequences. **NOTE: MAKE SURE YOUR PATTERN IS CONSISTENT TO AVOID MISCOUNTS**.

### Sorting files and getting unique counts
Now, lets imaging we have duplicate IDs. Let's create this scenario by concatenating twice the `example.fastq`, and redirecting the output to `double.fastq:


```python
cat example.fastq example.fastq > double.fastq
grep -c '^@M' double.fastq
```

This must give you twice as the number before. Now let's imagine we would like to have a list of IDs with unique identifyers. Fortuantely, bash have the right tool for it:


```python
grep '^@M' double.fastq | sort -u
```

The first part of the command `grep '^@M' double.fastq` will get the seqids as before, and the output would be piped into `sort -u`, which will sort the output, and return only the unique entries. Now let's imagine we dont want the entire output, but just the count, so our good old friend `wc -l` will come handy:


```python
grep '^@M' double.fastq | sort -u | wc -l
```

## Basic introduction to jobs in the cluster
Because a supercomputer or cluster is a collection of "blades" or individual computer connected to each other, there is a specific environment to make it work. For example, programs need to be installed in a way that would be accesible for every node. For this, Compute Canada uses the modules enviroment. This means that you will have to load the program (if available) that you would like to run. How can you know what is installed? well, there is a command for that:


```python
module avail
```

This command will give you a list of all available programs in the cluster. Normally, there are quite a bit of files and it is difficult to identify them all at once. So there is a second useful `module` command: `module spider`. Let's imagine we would like to use the command line version of blast. This suite is called blast+, so we can type:


```python
module spider blast+
```

This will show you all the entries and versions for blast+, along with some important information of pre-requisites you have to load before blast+. Lets take a look to them. In this case, it tells use that before we load blast+ we need the modules `nixpkgs/16.09`, and  `gcc/5.4.0`:


```python
module load nixpkgs/16.09 gcc/5.4.0 blast+
```

This will load the three modules we need. Let's see if it worked by typing `blastn -h`. Now we have the blast commands we need to run a job. However, **and this is very important**, DO NOT RUN JOBS ON THE LOGING NODE. If you do, you most likely will received a very angry email and your job will be killed before it finishes. This is done beacuse the login node is where eveybody launches their jobs to the rest of the computers, so by running something there it slows everybody down. To send jobs to the compute nodes you need `sbatch` and a launching script. I will get to that a bit later. Now, let's imagine that we need to do a smaller quick test on the spot. For that we will invoke the interactive shell using the `salloc` command. This command will allocate us some space in a compute node. To call this, we require the account that we are registered in compute canada. For Cristescu lab members it would be `def-mcristes`:


```python
salloc -A def-mcristes
```

where `-A` stands for account. Actually, you can also call it as `--account=def-mcristes`. Once it starts we can play around with the blast commands. This tutorial will cover only one basic nucleotide blast. Since we do not have downloaded the databses (extremely recomended if you do a lot of blasting), we will use the remote option. This option takes a bit more, for is ok for this tutorial:  


```python
blastn -db nt -query allponds.fasta -outfmt '6 sseqid qseqid evalue pident qcovs stitle' -remote -max_target_seqs 2
```

the `-db nt` tells the program to use the nucleotide database, the `-query allponds.fasta` is the query sequences we want to search against the NCBI datase, `-outfmt` is the format of the output and we put 6(tabular), and restricting the output to sseqid, qseqid, evalue, pident, qcovs, and stitle. The option `-remote` esentially tells the program to do it over the internet instead of with a particular database. The `-max_target_seqs 2` makes sure we retained only the top 2 entries of the search.

If you are interested in this tool, check out their manual at [NCBI](https://www.ncbi.nlm.nih.gov/books/NBK279690/).

Now to finish our session of `salloc`, type `exit`. This will bring us back to the login node. Now to actually use more demanding (both in memory and in time) tasks, we will need to submit a script to the cluster. Most of the basics are already covered in the Compute Canada wikis like [this one](https://docs.computecanada.ca/wiki/Running_jobs). First we need to create a script to launch the job, and it has a particular structure and keywords. Here is an example:
```
#!/bin/bash
#SBATCH --time=00:01:00
#SBATCH --account=def-mcristes
echo 'Hello, world!'
sleep 30
```
If you launch this script, it will send it to a computing node for no more than 1 minute. So let's create our own!! Copy this template, but replace the last two lines with something you would like. Also, try to redirect the output to a file named `myecho.txt`.

If what you are going to run is, for example, the previous blast run, the file should look like this:

```
#!/bin/bash
#SBATCH --time=00:10:00
#SBATCH --account=def-mcristes
module load nixpkgs/16.09 gcc/5.4.0 blast+
blastn -db nt -query allponds.fasta -outfmt '6 sseqid qseqid evalue pident qcovs stitle' -remote -max_target_seqs 2
```
See how the module call is included?


## WORK IT OUT!!
I have shown you the very basics! Now is time for you to do it on your own! 

1. Go to your home directory 
2. Create a folder (name it as you wish)
3. Go into that folder
4. Go to the NCBI website, select your favorite gene, in the make sure you selected FASTA (text). 
5. Use wget to download the sequence into your folder
6. Create a submission script and use blast remote as in the example, but with this query
7. Get the results in a file named myfirstclusterblast.txt

## Well, that is all for now, let's work a little bit on this and I'll take questions you might still have!!! cheers!


```python

```
