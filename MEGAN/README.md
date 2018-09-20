---


---

<h1 id="introduction-to-megan-methods">Introduction to MEGAN methods</h1>
<p>This tutorial (more like a conversation and notes), will try to introduce to some aspects of the Software <a href="http://www-ab.informatik.uni-tuebingen.de/software/megan6/">MEGAN</a>. I will assume that you are more or less familiar with metagenomics pipelines, and will focus more on how MEGAN does things.<br>
The papers and manual that you can check and where I got graphics and information are:</p>
<ul>
<li><a href="https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1004957">Huson et al.,MEGAN Community Edition - Interactive exploration and analysis of large-scale microbiome sequencing data,  PLoS Computational Biology, 2016, 12 (6): e1004957</a></li>
<li><a href="http://www.nature.com/nmeth/journal/v12/n1/full/nmeth.3176.html">Buchfink et al, Fast and sensitive protein alignment using DIAMOND, Nature Methods, 2015, 12:59-60.</a></li>
<li><a href="http://genome.cshlp.org/content/21/9/1552.long">Huson et al, Integrative analysis of environmental sequences using MEGAN4, Genome Res, 2011, 21:1552-1560.</a></li>
<li><a href="http://genome.cshlp.org/content/17/3/377.long">Huson et al, MEGAN analysis of metagenomic data, Genome Res, 2007, 17(3):377-86.</a></li>
<li><a href="https://www.google.com/url?sa=t&amp;rct=j&amp;q=&amp;esrc=s&amp;source=web&amp;cd=1&amp;cad=rja&amp;uact=8&amp;ved=2ahUKEwjinvOD6cTdAhXHxYMKHUy-AekQFjAAegQIBRAC&amp;url=http%3A%2F%2Fab.inf.uni-tuebingen.de%2Fdata%2Fsoftware%2Fmegan6%2Fdownload%2Fmanual.pdf&amp;usg=AOvVaw0RSRcHsae9qWupV1_G_ln6">User Manual for MEGAN V6.12.3</a></li>
</ul>
<p>There are also useful tutorials:</p>
<ul>
<li><a href="https://www.google.com/url?sa=t&amp;rct=j&amp;q=&amp;esrc=s&amp;source=web&amp;cd=2&amp;ved=2ahUKEwjcs5S86cTdAhVo9IMKHRfPA-cQFjABegQICRAC&amp;url=http%3A%2F%2Fab.inf.uni-tuebingen.de%2Fdata%2Fsoftware%2Fmegan6%2Fdownload%2FMeganTutorialApril2018.pdf&amp;usg=AOvVaw3hBaeMzeFdB-brR50gAXFq">Megan Tutorial 2018.key - Algorithms in Bioinformatics</a></li>
<li><a href="https://metagenomics-workshop.readthedocs.io/en/latest/annotation/taxonomic_annotation.html">Taxonomic annotation</a></li>
<li><a href="https://www.google.com/url?sa=t&amp;rct=j&amp;q=&amp;esrc=s&amp;source=web&amp;cd=14&amp;ved=2ahUKEwiGksOl6sTdAhWK3oMKHQfOCoYQFjANegQICRAC&amp;url=https%3A%2F%2Freadthedocs.org%2Fprojects%2Fmetagenomics-workshop%2Fdownloads%2Fpdf%2Flatest%2F&amp;usg=AOvVaw3x4abtP7EXWdmJloj9qZ14">Metagenomics Workshop SciLifeLab Documentation - Read the Docs</a></li>
</ul>
<h2 id="objectives--learning-outcomes">Objectives / learning outcomes:</h2>
<p>At the end of this tutorial you shoud be able to:</p>
<ol>
<li>Better understand the MEGAN pipeline</li>
<li>Underderstand and make use of DIAMOND</li>
<li>Know the file formats used in MEGAN</li>
<li>Understand the taxonomic assignment by LCA (last common ancestor)</li>
<li>Use command line script MEGANIZER to process data</li>
<li>Get a gist on the Long Read LCA algorithm: mainly longer contigs</li>
</ol>
<h2 id="outline-of-the-tutorial">Outline of the tutorial</h2>
<ol>
<li><a href="#the-megan-pipeline">The MEGAN pipeline: a general overview</a></li>
<li><a href="#diamond-a-lighting-fast-blast">DIAMOND: a lighting fast blast</a></li>
<li><a href="#meganizer-from-.daa-to-megan">Meganizer: from .daa to MEGAN</a></li>
<li><a href="#lca-taxonomic-binning">LCA taxonomic binning</a></li>
<li><a href="meganserver-Connecting-a-server-and-the-app">MEGANServer: Connecting a server and the app</a></li>
</ol>
<h2 id="the-megan-pipeline">The MEGAN pipeline</h2>
<p><img src="https://journals.plos.org/ploscompbiol/article/figure/image?size=large&amp;id=10.1371/journal.pcbi.1004957.g001" alt="MEGAN pipeline"></p>
<p>Overall, MEGAN pipeline can be divided in 4 steps:</p>
<ol>
<li>The input files (already QCed) are analyzed with Diamond (we will get to this in a sec)</li>
<li>Diamond (or blast) outputs are converted into MEGAN DAA files</li>
<li>Storage in server</li>
<li>Interactive inteaction</li>
</ol>
<p>Basically, you align your reads to a database, bin your reads with annotations, and finally visualize the results. Given that most of your are comfortable with the graphic interface of MEGAN, I will focus on the first three steps.</p>
<h2 id="diamond-a-lighting-fast-blast">DIAMOND: a lighting fast blast</h2>
<p>DIAMOND stands for <strong>D</strong>ouble <strong>I</strong>ndex <strong>A</strong>lign<strong>M</strong>ent <strong>O</strong>f <strong>N</strong>ext-generation sequencing <strong>D</strong>ata, and you can get it <a href="http://ab.inf.uni-tuebingen.de/software/diamond">here</a>, and if you haven’t install it now! They report that DIAMOND is ~20,000 times faster than BLASTX. DIAMOND, as MEGAN, are designed for metagenomics, particularly to explore species/OTUs profile, and therefore is usual to align reads to protein databases (hence usage blastx).</p>
<p>To understand the underlying algorithm of DIAMOND, we need to compare with the traditional blast. Both used the seed and extend strategy (images from <a href="https://www.doi.org/10.4172/jcsb.1000260">Alarfaj et al.</a>):</p>
<h5 id="seed">SEED</h5>
<p><img src="https://raw.githubusercontent.com/jshleap/CristescuLab_misc/master/Tutorials/Blast/img/Seeding-step.png" alt="enter image description here"></p>
<h5 id="extend">EXTEND</h5>
<p><img src="https://raw.githubusercontent.com/jshleap/CristescuLab_misc/master/Tutorials/Blast/img/extension_step.png" alt="enter image description here"></p>
<p>However, DIAMOND modifies the seeding in two ways:</p>
<ol>
<li>Reduced alphabet: Their empirical studies allowed them to “re-write” the protein alphabet. This rewrite is done by grouping aminoacids into 11 categories: [KREDQN] [C] [G] [H] [ILV] [M] [F] [Y] [W] [P] [STA].</li>
<li>Spaced seeds: Allowing for different “shapes” of seeds and spaces within each seed.<br>
<img src="https://media.nature.com/lw926/nature-assets/nmeth/journal/v12/n1/images_supplementary/nmeth.3176-SF1.jpg" alt="enter image description here"></li>
<li>Seed index: In short (lots of CS jargon in this), the query is pre-processed and check for common seeds. Each grouped seed is associated with the query names that contains it so that the access during the alignments is faster.</li>
<li>Double indexing: Again, lots of CS jargon, but putting symply seeds from subject and query are both index and sorted, making the matching very fast.<br>
<img src="http://pangenome.tuebingen.mpg.de/images/diamond.png" alt="enter image description here"></li>
</ol>
<p>More details of the DIAMOND algorithm can be found in <a href="http://ab.inf.uni-tuebingen.de/teaching/ws14/bioinf/13-BeyondBlast.pdf">this very nice tutorial</a>, and I encourage everybody to read it and bring doubts to other tutorials. From that tutorial:</p>
<blockquote>
<p>In summary, here is an outline of the DIAMOND algorithm.</p>
<ol>
<li>Input the list of query sequences <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>Q</mi></mrow><annotation encoding="application/x-tex">Q</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.8777699999999999em; vertical-align: -0.19444em;"></span><span class="mord mathit">Q</span></span></span></span></span>.</li>
<li>Translate all queries, extract all spaced seeds and their locations, using a reduced alphabet.</li>
<li>Sort them. Call this <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>S</mi><mo>(</mo><mi>Q</mi><mo>)</mo></mrow><annotation encoding="application/x-tex">S(Q)</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord mathit" style="margin-right: 0.05764em;">S</span><span class="mopen">(</span><span class="mord mathit">Q</span><span class="mclose">)</span></span></span></span></span>.</li>
<li>Input the list of reference sequences <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>R</mi></mrow><annotation encoding="application/x-tex">R</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span><span class="mord mathit" style="margin-right: 0.00773em;">R</span></span></span></span></span>.</li>
<li>Extract all spaced seeds and their locations, using a reduced alphabet. Sort them. Call this <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>S</mi><mo>(</mo><mi>R</mi><mo>)</mo></mrow><annotation encoding="application/x-tex">S(R)</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord mathit" style="margin-right: 0.05764em;">S</span><span class="mopen">(</span><span class="mord mathit" style="margin-right: 0.00773em;">R</span><span class="mclose">)</span></span></span></span></span>.</li>
<li>Traverse <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>S</mi><mo>(</mo><mi>Q</mi><mo>)</mo></mrow><annotation encoding="application/x-tex">S(Q)</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord mathit" style="margin-right: 0.05764em;">S</span><span class="mopen">(</span><span class="mord mathit">Q</span><span class="mclose">)</span></span></span></span></span>. and <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>S</mi><mo>(</mo><mi>R</mi><mo>)</mo></mrow><annotation encoding="application/x-tex">S(R)</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord mathit" style="margin-right: 0.05764em;">S</span><span class="mopen">(</span><span class="mord mathit" style="margin-right: 0.00773em;">R</span><span class="mclose">)</span></span></span></span></span> simultaneously. For each seed S that occurs both in <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>S</mi><mo>(</mo><mi>Q</mi><mo>)</mo></mrow><annotation encoding="application/x-tex">S(Q)</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord mathit" style="margin-right: 0.05764em;">S</span><span class="mopen">(</span><span class="mord mathit">Q</span><span class="mclose">)</span></span></span></span></span> and <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>S</mi><mo>(</mo><mi>R</mi><mo>)</mo></mrow><annotation encoding="application/x-tex">S(R)</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 1em; vertical-align: -0.25em;"></span><span class="mord mathit" style="margin-right: 0.05764em;">S</span><span class="mopen">(</span><span class="mord mathit" style="margin-right: 0.00773em;">R</span><span class="mclose">)</span></span></span></span></span>, consider all pairs of its locations x and y in <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>Q</mi></mrow><annotation encoding="application/x-tex">Q</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.8777699999999999em; vertical-align: -0.19444em;"></span><span class="mord mathit">Q</span></span></span></span></span> and <span class="katex--inline"><span class="katex"><span class="katex-mathml"><math><semantics><mrow><mi>R</mi></mrow><annotation encoding="application/x-tex">R</annotation></semantics></math></span><span class="katex-html" aria-hidden="true"><span class="base"><span class="strut" style="height: 0.68333em; vertical-align: 0em;"></span><span class="mord mathit" style="margin-right: 0.00773em;">R</span></span></span></span></span>, respectively: check whether there is a left-most seed match. If this is the case, then attempt to extend the seed match to a significant banded alignment.</li>
</ol>
</blockquote>
<p>Now, let’s compare how DIAMOND does against BLAST. To do so, we need to format the databases both for blastx, and for DIAMOND. I assume you are versed in blast command line tools, and if not, I encourage you to read the <a href="https://github.com/jshleap/CristescuLab_misc/tree/master/Tutorials/Blast">tutorial</a> we did a couple of sessions ago. If you are part of the Cristescu Lab, the pre-formated diamond database for nucleotides can be found in our database folder under NCBI. If you are not, please download <a href="ftp://ftp.ncbi.nlm.nih.gov/blast/db/FASTA/nr.gz">this file</a> (unfortunately it takes a while and 45Gb). Wether you are part of the CristescuLab or not, format a database (for the Cristescu lab, use a mock fasta file) with the <code>diamond makeblastdb</code> command:</p>
<pre><code>cd PATH/TO/FASTA
diamond makedb --in &lt;FASTA_FILE&gt; -d &lt;NAME_DB&gt;
</code></pre>
<p>Replace PATH/TO/FASTA, FASTA_FILE, and NAME_DB with your actual values. In the CristescuLab <code>~/projects/def-mcristes/Databases/NCBI</code>, I have pre-formatted the full nt database by downloading the fasta file of the entire database (and hence the 45Gb) and executed:<br>
<code>zcat nr.gz| diamond makedb -d diamond_nr</code></p>
<p>Once we have done this, we are ready for the test with <a href="https://raw.githubusercontent.com/jshleap/CristescuLab_misc/master/Tutorials/MEGAN/files/sample.fa">this mock fasta file</a>.</p>
<p>Let’s now run a blastx with both BLAST and DIAMOND algorithms (<strong><em>remember to load them or download them</em></strong>):<br>
<code>time blastx -db PATH/TO/DATABASE/nr -query /PATH/TO/QUERY/sample.fa -out &lt;OUTFN&gt; -num_threads &lt;CPUS&gt;</code><br>
<strong><em>Just kidding don’t run it… it will take several hours. If you want you can download the results <a href="https://raw.githubusercontent.com/jshleap/CristescuLab_misc/master/Tutorials/MEGAN/files/submitted.blast">here</a></em></strong><br>
This one you can run though! <strong>Make sure to use a salloc with at least 32 cpus</strong><br>
<code>time diamond blastx -d PATH/TO/DATABASE/diamond_nr -q /PATH/TO/QUERY/sample.fa -f 100 --salltitles -o &lt;OUTFN&gt;.daa</code><br>
Or you can just get the result <a href="https://raw.githubusercontent.com/jshleap/CristescuLab_misc/master/Tutorials/MEGAN/files/diamond.daa">here</a>.<br>
Now we have a .daa file (-f is the format, 100 is the binary output format) which basically is the DIAMOND output.</p>
<h5 id="but-not-everything-shines">But not everything shines!</h5>
<p>In my experience the accuracy drops significantly even with the more sensitive approach. Not great for accurate functional annotation when false positives want to be controlled.</p>
<h2 id="meganizer-from-.daa-to-megan">Meganizer: from .daa to MEGAN</h2>
<p>After we have a DIAMOND binary output format (MEGAN call it .daa). This file format is DIAMOND proprietary format, but MEGAN supports it. Before we dig into Meganizer, lets take a peak of this file with DIAMOND:<br>
<code>diamond view --daa OUTFN.daa | head</code></p>
<p>The MEGAN user interface uses a different compressed file format called RMA. This is essentially a compressed and comprehensive file that summarizes annotation and classifications to be visualized with the MEGAN GUI. This step takes the information from the alignment to a database (either DIAMOND or BLAST), the taxonomic information, and functional information, and append them in a single file in a relational manner that MEGAN-CE can understand. We can do this either with <code>daa-meganizer</code> or with <code>daa2rma</code>, but we need mapping files between the reference database and the functional annotations. For this example, lets focus in the KEGG annotation, and the mapping file can be downloaded <a href="http://ab.inf.uni-tuebingen.de/data/software/megan6/download/acc2kegg-Dec2017X1-ue.abin.zip">here</a>, and the taxonomic mapping can be found <a href="http://ab.inf.uni-tuebingen.de/data/software/megan6/download/prot_acc2tax-June2018X1.abin.zip">here</a>. These files need to be unzipped. Now let’s check some of the options first:<br>
<code>PATH/TO/daa2rma -h</code></p>
<p>Therefore if we want to include the KEGG annotations we will do:<br>
<code>PATH/TO/daa2rma -i OUTFN.daa -o reads.rma -a2t prot_acc2tax-June2018X1.abin -a2kegg acc2kegg-Dec2017X1-ue.abin.zip -fun KEGG</code></p>
<h3 id="lca-taxonomic-binning">LCA taxonomic binning</h3>
<h2 id="meganserver-connecting-a-linux-server-and-the-app">MEGANServer: Connecting a Linux server and the app</h2>

