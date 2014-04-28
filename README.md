assemblage [əˈsɛmblɪdʒ]  
*noun*

1. a number of things or persons assembled together; collection; assembly
2. (Cookery) a list of dishes served at a meal or the dishes themselves
3. the act or process of assembling or the state of being assembled
4. (Fine Arts & Visual Arts / Art Terms) a three-dimensional work of art that combines various objects into an integrated whole

From Collins English Dictionary – Complete and Unabridged © HarperCollins Publishers 1991, 1994, 1998, 2000, 2003

Update:
======

As of 2013-10-01, the scripts for making blobplots / Taxon-annotated GC-coverage plots (TAGC plots) are being moved and maintained at [github.com/blaxterlab/blobology](github.com/blaxterlab/blobology)

The scripts in this repository are being left as they are, as a record of the work accompanying my PhD thesis.

About
=====

This is a set of scripts for working with genome assemblies, making taxon-annotated GC-coverage plots (a.k.a blob plots), extracting reads belonging to the blobs, etc

These blob plots as used here were described in http://www.ncbi.nlm.nih.gov/pmc/articles/PMC3294205/

Only 6 months after this paper was published did our group realise that taxon-annotated GC-coverage plots acronyms to, wait for it, TAGC plots! (thank Mark Blaxter for spotting that).

Figure 3 from the paper:

![Caenorhabditis sp 5 Blob Plot](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC3294205/bin/13199_2012_154_Fig3_HTML.jpg)

Figure 3: A low-complexity metagenome. Taxon-annotated GC-coverage scatter-plot (as in Fig. 2) for contigs from a preliminary SE assembly of a Caenorhabditis sp 5 DNA sample. 10,000 randomly selected contigs were BLAST-annotated by comparison to the NCBI nt database, and coloured by class-of-origin of the best match identified. Full colour version is available online

There are three other READMEs in this repository corresponding to three of the chapters of my thesis:

* [README-annotation.md](https://github.com/sujaikumar/assemblage/blob/master/README-annotation.md)
* [README-CNE.md](https://github.com/sujaikumar/assemblage/blob/master/README-CNE.md)
* [README-meloidogyne.md](https://github.com/sujaikumar/assemblage/blob/master/README-meloidogyne.md)


Example Data Sets
=================

As an example, we can use the following study from the Short Read Archive: 

* ERP001495 - De novo whole-genome sequence of the free-living nematode Caenorhabditis sp. 5 strain JU800 DRD-2008

This study has one sample:

* ERS147916 - Caenorhabditis sp. 5 JU800 DRD-2008  

Two libraries (or "Experiments" in SRA terminology) were run for this sample:

* ERX114449 - 300 bp library with Illumina HiSeq2000 101 bp PE sequencing
* ERX114450 - 600 bp library with Illumina HiSeq2000 101 bp PE sequencing

And these are the four files that can be accessed from http://www.ebi.ac.uk/ena/data/view/ERP001495

* `g_ju800_110714HiSeq300_1.txt.gz` - 300 bp library forward read
* `g_ju800_110714HiSeq300_2.txt.gz` - 300 bp library reverse read
* `g_ju800_110714HiSeq600_1.txt.gz` - 600 bp library forward read
* `g_ju800_110714HiSeq600_2.txt.gz` - 600 bp library reverse read

How-Tos
=======

This list of How-Tos demonstrates how the scripts in this repository can be used in conjunction with other installed software to assemble nematode (and other small metazoan genomes).

How to make a taxon-annotated GC cov "blob" plot
------------------------------------------------

This section is a meta section that describes the How-Tos you need to make a taxon-annotated GC-cov blob plot like the one at http://www.ncbi.nlm.nih.gov/pmc/articles/PMC3294205/figure/Fig3/

1. adapter- and quality-trim Illumina fastq reads using sickle and scythe in one command with no intermediate files
2. create a preliminary assembly using ABySS
3. map reads to an assembly to get insert-size and coverage information
4. assign high-level taxon IDs to contigs
5. make blobology plots with R

How to sub-sample sequences at random for test read sets
--------------------------------------------------------

The example read files are between 5 and 8 GB each, and processing the complete data set could take a long time. If you just want to run all the How-Tos in this README on smaller files to test how the scripts work, use the `pick_random.pl` script:

    zcat g_ju800_110714HiSeq300_1.txt.gz | pick_random.pl 4 0.1 > g_ju800_110714HiSeq300_1.txt.r0.1

This command will gunzip the gzipped fastq file (`zcat file.gz` is the same as `gunzip -c file.gz`) and pipe the output through `pick_random.pl`. The script takes two arguments - the first tells you how many lines at a time should be treated as a block, while the second tells you what proportion of the file to pick (0.1 or 10% in this case)

However, running this command independently on the forward (`_1`) and reverse (`_2`) read files will result in different reads being picked. To sample both pairs at the same time, first interleave the files, and then pick 8 lines at a time (4 for the forward read, and 4 for the reverse read), and then use `pick_random.pl` to get 10% of the reads:

    shuffleSequences_fastx.pl 4 <(zcat g_ju800_110714HiSeq300_1.txt.gz) <(zcat g_ju800_110714HiSeq300_2.txt.gz`) |
    pick_random.pl 8 0.1 > g_ju800_110714HiSeq300_interleaved.txt.r0.1

Note: The <() syntax is for bash process substitution. Anything inside <(...) will be executed and its output will be treated by the containing command as if it is a file.

`shuffleSequences_fastx.pl` is based on shuffleSequences_fastq.pl that used to ship with Velvet. The difference is that it can be used to shuffle both fasta (set 1st argument to 2) and fastq (set 1st argument to 4) sequences.

`g_ju800_110714HiSeq300_interleaved.txt.r0.1` is an interleaved/shuffled file. If you want the _1 and the _2 reads in separate files, you will have to do one more step:

    unshuffleSequences_fastx.pl 4 g_ju800_110714HiSeq300_interleaved.txt.r0.1

which will create `g_ju800_110714HiSeq300_interleaved.txt.r0.1.1` and `g_ju800_110714HiSeq300_interleaved.txt.r0.1.2`

How to use FastQC to check raw read qualities before assembly
-------------------------------------------------------------

Run FastQC (we used version 0.10.0) using this command:

    fastqc --nogroup <readfiles...>

* --nogroup shows each cycle separately rather than grouping cycle 10 onwards in groups of 5. Very useful for pinpointing failed cycles (i.e. cycles with lots of Ns)    

FastQC output is divided into sections as described below:

* Basic Statistics: Total number of reads and read length. A rough estimate of the genome coverage is given by multiplying the number of reads by the read length to get the number of bases sequenced per file, summing across all files, and dividing this number by the expected genome size. If there are too few reads (<50X genome coverage), more sequencing will almost certainly be needed for assembling a de novo genome.
* Per base sequence quality: Low quality towards the end of each read is expected and can be trimmed. Typically, a Phred quality threshold of 20 is used. Most cycles/positions should have a median quality above 20, otherwise the run may be very low quality and will assemble poorly. If there is a quality dip one or more times in the middle of the sequence (or at the start), then additional correcting might be needed. Most quality trimming algorithms use sliding windows and trim only from the 3' end.
* Per sequence quality scores - Mean sequence quality histogram
* Per base sequence content and per base GC content: Genomic DNA should have roughly even levels of G and C, and roughly even levels of A and T. If the first few bases are very skewed, then that might indicate that multiplexing barcode adapters have been left on. G > C, or A > T might indicate sequencing bias, a known problem with Illumina sequencing. For RNA-Seq assemblies, the first 10 bases will often be even more skewed or "bumpy", but that is expected as RNA-Seq samples are primed with random hexamers nonamers which are a) not equally distributed in the original hexamer or nonamer mix and b) anneal at different rates, further increasing the skew.
* Per sequence GC content - An uncontaminated genome sample will typically have a single GC peak. If there is a bump, that might indicate a contaminant in the sample. Typically, for nematodes that have an average GC between 30-50%, a bump on the right slope might indicate bacterial contamination, as many bacteria have higher GC content.
* Per base N content: A large number of Ns at any one position might indicate an Illumina cycle failure. Most assembly algorithms throw away reads with an N in them, but if the proportion of reads thrown away is too large, error correction might be a better solution.
* Sequence Length Distribution - Nothing to check here for raw Illumina reads - all reads are the same length. However, this plot could be useful for adapter- and quality- trimmed reads to see how long the remaining good quality reads are.
* Sequence Duplication Levels - expected to be high for high coverage, and for pcr duplications in mate-pair data.
* Overrepresented sequences. If there are any adapter sequences, they usually show up here. RNA-Seq libraries may also show overrepresented reads (or k-mers, see below) because of highly expressed genes in the sample.
* K-mer Content - Hint at adapter sequences present (for example if the library insert is smaller than the read length) or if the multiplexing barcode is still present at the start of the read. We found 8-mers more useful than the default 5-mers, and you can modify the FastQC source code to change the default length.

How to adapter- and quality-trim Illumina fastq reads using sickle and scythe in one command with no intermediate files
-----------------------------------------------------------------------------------------------------------------------

(Hmm, that's a bit of a long How-To title, almost like article titles in the 1600s, such as this one: [An Observation and Experiment Concerning a Mineral Balsom, Found in a Mine of Italy by Signior Marc-Antonio Castagna; Inserted in the 7th. Giornale Veneto de Letterati of June 22. 1671, and Thence English'd as Follows](http://dx.doi.org/10.1098/rstl.1671.0068))

Requirements:

1. Install scythe for adapter trimming from https://github.com/ucdavis-bioinformatics/scythe 
2. Install sickle for quality trimming from https://github.com/ucdavis-bioinformatics/sickle
3. Install GNU Parallel (optional, but highly recommended!) from http://www.gnu.org/software/parallel/
4. A file with Illumina adapters - you can use `adapters.fa` from this repository

Command:

    sickle pe -t sanger -n -l 50 \
        -f <(scythe -a adapters.fa <(zcat g_ju800_110714HiSeq300_1.txt.gz) -q sanger \
            2> g_ju800_110714HiSeq300_1.scythe.err | perl -plne 's/^$/A/; s/ 1.*/\/1/')  \
        -r <(scythe -a adapters.fa <(zcat g_ju800_110714HiSeq300_2.txt.gz) -q sanger \
            2> g_ju800_110714HiSeq300_2.scythe.err | perl -plne 's/^$/A/; s/ 2.*/\/2/')  \
        -o >(gzip >g_ju800_110714HiSeq300_1.clean.txt.gz) \
        -p >(gzip >g_ju800_110714HiSeq300_2.clean.txt.gz) \
        -s >(gzip >g_ju800_110714HiSeq300_s.clean.txt.gz) \
            &>g_ju800_110714HiSeq300.sickle.err
        
The command above will first run scythe to search for adapters.fa in `g_ju800_110714HiSeq300_1.txt.gz` and `g_ju800_110714HiSeq300_2.txt.gz` simultaneously. The stderr stream of scythe is stored in `g_ju800_110714HiSeq300_1.scythe.err` and the output of scythe is parsed through a perl one liner that replaces blank lines with a single A, and adds "/1" or "/2" to the read header. This perl one liner is needed because scythe screws up the read names, and because the next script sickle can't deal with blank sequence lines where the whole sequence has been adapter-trimmed.

`sickle pe -t sanger -n -l 50` then runs sickle on the files output by the scythe and perl one liner above:

* `pe` means treat corresponding reads from the `-f` and `-r` files as being parts of a pair
* `-t sanger` tells it that the quality values are in sanger fastq encoding
* `-n` discards reads/pairs with any Ns in them
* `-l 50` discards reads/pairs with fewer than 50 bp
* `-o` filename of the output forward read
* `-p` filename of the output paired reverse read
* `-s` filename of the singletons (where the other read in the pair was discarded if it had an N or was shorter than `-l 50`

Additional scythe and sickle parameters can be set as needed (minimum number of matches to adapter sequence, trim bases from start of read, etc).

If you do have GNU Parallel installed, you can run multiple libraries at the same time like this:

    parallel "sickle pe -t sanger -n -l 50 \
        -f <(scythe -a adapters.fa <(zcat {}_1.txt.gz) -q sanger \
            2> {}_1.scythe.err | perl -plne 's/^$/A/; s/ 1.*/\/1/')  \
        -r <(scythe -a adapters.fa <(zcat {}_2.txt.gz) -q sanger \
            2> {}_2.scythe.err | perl -plne 's/^$/A/; s/ 2.*/\/2/')  \
        -o >(gzip >{}_1.clean.txt.gz) \
        -p >(gzip >{}_2.clean.txt.gz) \
        -s >(gzip >{}_s.clean.txt.gz) \
            &>{}.sickle.err" ::: g_ju800_110714HiSeq300 g_ju800_110714HiSeq600 

This command will create three files for each library:
 - `g_ju800_110714HiSeq300_1.clean.txt.gz`
 - `g_ju800_110714HiSeq300_2.clean.txt.gz`
 - `g_ju800_110714HiSeq300_s.clean.txt.gz`
 - `g_ju800_110714HiSeq600_1.clean.txt.gz`
 - `g_ju800_110714HiSeq600_2.clean.txt.gz`
 - `g_ju800_110714HiSeq600_s.clean.txt.gz`

You can interleave the _1 and _2 files:

    shuffleSequences_fastx.pl 4 \
        <(zcat g_ju800_110714HiSeq300_1.clean.txt.gz) \
        <(zcat g_ju800_110714HiSeq300_2.clean.txt.gz) |
    gzip > g_ju800_110714HiSeq300.clean.txt.gz

How to create a preliminary assembly using ABySS
------------------------------------------------

Requirements:

1. Install ABySS from http://www.bcgsc.ca/platform/bioinfo/software/abyss . This README was tested with versions 1.3.3 and 1.3.4. Versions before 1.3.3 did not work as well. It is possible that newer versions will have different options and work even better.

To get a preliminary assembly (PASS) with no pairing information (i.e. treating all reads as single-end) from the cleaned reads above:

    name=C20rp.31.n10.se;
    mkdir $name;
    abyss-pe -C $name name=$name n=10 k=31 \
        se='g_ju800_110714HiSeq300.clean.txt.gz \
            g_ju800_110714HiSeq600.clean.txt.gz' &>$name/log

Making a new directory with the parameters used makes it easy to track the settings used if we later create a lot of assemblies with different settings.  

* `-C` puts the output in the specified directory (`$name` in this case) rather than in the current directory
* `n=10` is the minimum number of connecting pairs needed before two unitigs or contigs are joined
* `k=31` is the k-mer used for making the de Bruijn graph. 31 is a reasonable default. Longer k-mers may give more contiguous assemblies but the goal of the PASS is to get as much assembled sequence as possible rather than get the longest contigs.

The single-end PASS will be in `$name/$name-contigs.fa`

How to create a preliminary assembly using CLC
----------------------------------------------

Install the CLC Assembly Cell suite of tools (this README was tested with version 4.06 beta). For a single end assembly, simply provide all error-corrected read files to `clc_novo_assemble`:

    clc_novo_assemble -o clcse.fna -q \
    g_ju800_110714HiSeq300_1.clean.txt.gz g_ju800_110714HiSeq300_2.clean.txt.gz \
    g_ju800_110714HiSeq600_1.clean.txt.gz g_ju800_110714HiSeq600_2.clean.txt.gz

 - `-o` is the output assembly fasta file
 - `-q` is the option after which all read files should be included

See the manual for `clc_novo_assemble` at http://www.clcbio.com/index.php?id=1393&manual=A_6_Options_clc_novo_assemble.html

How to map reads to an assembly to get insert-size and coverage information using Bowtie 2
------------------------------------------------------------------------------------------

1. Install Bowtie 2 from http://bowtie-bio.sourceforge.net/bowtie2 . This README was tested with Version 2.0.0-beta5 and Version 2.0.0-beta6 but will probably work with newer versions as well if the syntax has not changed.
2. Install samtools from http://samtools.sourceforge.net
3. `sam_len_cov_gc_insert.pl` from this repository
4. Preliminary assembly - a fasta file with contigs `filename-contigs.fa`.
5. Interleaved read files for each library. They do not have to be interleaved to calculate coverage for each contig, but if they are interleaved, then generating insert-size plots (next How-To) is easier. In this example, I assume that we want to map the trimmed 300 bp and the 600 bp interleaved files:
 - `g_ju800_110714HiSeq300.clean.txt.gz`
 - `g_ju800_110714HiSeq600.clean.txt.gz`

Create a bowtie2 index

    assemblyfile=filename-contigs.fa
    bowtie2-build $assemblyfile $assemblyfile

Map reads for multiple libraries using a bash for loop

    for lib in g_ju800_110714HiSeq300.clean.txt.gz g_ju800_110714HiSeq600.clean.txt.gz
    do
        bowtie2 -x $assemblyfile -q -U $lib --very-fast-local -p 8 --reorder --mm | 
        tee >(samtools view -S -b - > $assemblyfile.$lib.bowtie2.bam) |
        tee >(samtools view -S -b - | samtools sort -m 2000000000 - $assemblyfile.$lib.bowtie2.sorted) |
        sam_len_cov_gc_insert.pl -i -f $assemblyfile -s - -out $assemblyfile.$lib
    end

The two `tee` commands take the SAM output generated by bowtie2 and write them out as read-sorted `$assemblyfile.$lib.bowtie2.bam` and contig-sorted `$assemblyfile.$lib.bowtie2.sorted.bam`.

`sam_len_cov_gc_insert.pl` takes the following options:

* `-i` use this switch if your read file is interleaved and you want to estimate insert sizes. Leave it out if you used bowtie2 without an interleaved file
* `-f` assembly fasta file
* `-s` alignments in SAM format
* `-out` output file prefix

and creates the following files:

* `outputprefix.lencovgc.txt` - tab delimited text file with these columns:
 1. library name (read filename by default)
 2. contig id
 3. contig length
 4. contig coverage (read coverage, not k-mer coverage)
 5. contig gc content (proportion from 0 to 1)
 
 This file is used to make taxon-annotated GC-coverage "blob" plots
 
* `outputprefix.lencovgc.fna` - a fasta file (with each sequence in a single line) where the contig id has a useful suffix so the fasta header looks like this `>contigid_len_cov_gc`, e.g., `>contig23_1221_67.623_0.3222349`. We found this file useful for quickly pulling out subsets of contigs that met a certain GC and coverage criteria, as described in **How to select preliminary assembly (PASS) contigs with given GC or coverage**.

If you used interleaved read files and the `-i` switch in `sam_len_cov_gc_insert.pl`, then you also get two additional files:

* `outputprefix.pairing.hist.txt` - Frequency distribution of each insert size and whether the reads were pointed at each other ("innies") or away from each other ("outies", as you would expect from Illumina's mate-pair protocol)
* `outputprefix.pairing.stat.txt` - Some statistics about what percentage of reads were mapped to the same contig and how many were innies or outies. etc.

How to map reads to an assembly to get insert-size and coverage information using CLC
-------------------------------------------------------------------------------------

Requirements

1. CLC's Assembly Cell suite in your path. Will be using two tools:
 - `clc_ref_assemble`
 - `assembly_info`
2. A preliminary assembly (will most likely be a single-end assembly if made with `clc_novo_assemble`. eg `clcse.fna`
3. `clc_len_cov_gc_insert.pl` from this repository.

If you have separate forward and reverse reads, use the -i option to map them back like this:

    clc_ref_assemble -d clcse.fna -o clcse.fna.lib300_interleaved.cas \
    -q -i g_ju800_110714HiSeq300_1.clean.txt.gz g_ju800_110714HiSeq300_2.clean.txt.gz

- `-d` is the reference fasta file (in this case, a preliminary assembly)
- `-o` gives the output file (in CLC's .cas format)

Note that although we are using -i, we are not using -p etc because we don't want the mapper to assume a pairing insert size for now.

If you have multiple libraries, and interleaved files for each, run each mapping separately:

    clc_ref_assemble -d clcse.fna -o clcse.fna.lib300_interleaved.cas \
    -q -i g_ju800_110714HiSeq300_1.clean.txt.gz g_ju800_110714HiSeq300_2.clean.txt.gz
    clc_ref_assemble -d clcse.fna -o clcse.fna.lib600_interleaved.cas \
    -q -i g_ju800_110714HiSeq600_1.clean.txt.gz g_ju800_110714HiSeq600_2.clean.txt.gz

The important thing is to have separate .cas files for each library

Create len cov gc tables and insert size tables so that R can plot them.

Do this for each lib .cas file created separately in the step above:

    clc_len_cov_gc_insert.pl -c clcse.fna.lib300_interleaved.cas -i -o lib300 -lib lib300
    clc_len_cov_gc_insert.pl -c clcse.fna.lib600_interleaved.cas -i -o lib600 -lib lib600

The `-i` option calculates insert sizes, but only works if `clc_ref_assemble` was run with INTERLEAVED input (either with an interleaved FASTA file or with -q -i).

This script looks inside the cas file, gets the "reference" file (e.g., the clc se assembly), and gets length and GC from this file. Let's say this is called clcse.fna

Then, it uses `assembly_info clcse.fna.lib300_interleaved.cas` to get the read (not k-mer) coverage of each contig, and spits out a file called `clcse.fna.lib300_interleaved.cas.lencovgc.txt` (in the same location as `clcse.fna.lib300_interleaved.cas`) which is a tab delimited file with the following headers:

    read_set    contig_id   len     cov     gc

It also creates a copy of the reference fasta file where the header has been modified to:

    >contig_id_<length>_<readcoverage>_<gc>

If the `-i` option was used, these additional files are created with information about insert size estimates:

    clcse.fna.lib300_interleaved.cas.insert.freq.txt
    clcse.fna.lib300_interleaved.cas.insert.stat.txt

`clcse.fna.lib300_interleaved.cas.insert.freq.txt` is a tab delimited file without headers that has these columns:
    
    read_set    type(FR/RF)     insert_size     freq
    
Options for the clc_len_cov_gc_insert.pl script let you specify:
- `-o` outfile (a new output filepath rathe rhan the default which is the .cas filepath)
- `-l` libname (a new libname rather than the default, which is the .cas file name (eg `clcse.fna.lib300_interleaved.cas`))
- `-f` fastafile (a different, manually specified reference fasta file rather than the default - which is the reference file in `clcse.fna.lib300_interleaved.cas`)
- `-d` to change the delimited in the fasta header (default "_")

Warning: the script gets the path of the reference file from the .cas file. So if you ran `clc_ref_assemble` with relative paths, it will expect the reference at those relative locations.

If it doesn't find the files, you can either:

1. run `clc_len_cov_gc_insert.pl` from the same location that you ran `clc_ref_assemble` from (easiest)
2. use the `-f` option in `clc_len_cov_gc_insert.pl` to manually specify the reference fasta location (but be careful that this is correct, else the script won't give errors and you'll end up with weird len cov gc stats
3. modify the file paths in `clcse.fna.lib300_interleaved.cas` using clc's `change_assembly_files` utility
4. rerun `clc_ref_assemble` with absolute filepaths (takes long, don't bother)

This generates length, coverage, GC content, and insert size statistics for each sample in the following output files:

- lib300.insert.freq.txt : tab delim: read_set, type(FR/RF), insert_size, freq
- lib300.insert.stat.txt : stats about total pairs, %FR, %RF, %FF etc
- lib300.lencovgc.txt    : tab delim: read_set, contig_id, len, cov, gc
- lib300.lencovgc.fna    : same as assembly fasta, but with seqid formatted as `>contig_id_len_cov_gc` 


How to make a plot of insert sizes for each library
---------------------------------------------------

If you used interleaved reads when mapping the read files to the preliminary assembly using bowtie2 or CLC (in the previous How-Tos), then you can use the following R script to create a pretty plot of the insert sizes.

Requirements:

1. R installed (Rscript in your path)
2. R packages ggplot2 and grid installed
3. `plot_insert_freq_txt_binned.R` from this repository
4. `outputprefix.pairing.hist.txt` from the output of `sam_len_cov_gc_insert.pl` with `-i` option (previous How-To)

    plot_insert_freq_txt_binned.R outputprefix.pairing.hist.txt

How to assign high-level taxon IDs to contigs
---------------------------------------------

Requirements

1. NCBI blast+ suite installed from ftp://ftp.ncbi.nih.gov/blast/executables/blast+/LATEST
2. NCBI nt blast db downloaded and unpacked from ftp://ftp.ncbi.nih.gov/blast/db/
 * `wget ftp://ftp.ncbi.nih.gov/blast/db/nt.??.tar.gz`
3. NCBI taxonomy dumps downloaded and unpacked:
 * `wget ftp://ftp.ncbi.nih.gov/pub/taxonomy/gi_taxid_nucl.dmp.gz`
 * `wget ftp://ftp.ncbi.nih.gov/pub/taxonomy/taxdump.tar.gz`
4. `fastaqual_select.pl` from this repository
5. `blast_taxonomy_report_topscore.pl` from this repository
 
I'm using bash variables to store filenames so that the command syntax is more generic:

    assemblyfile=filename-contigs.fa
    
Select some contigs at random from the preliminary assembly:

    fastaqual_select.pl -f $assemblyfile -s r -n 10000 >$assemblyfile.r10000

* `-f` is the name of the fasta file
* `-s r` sorts at random
* `-n 10000` picks 10000 contigs. you can pick more or less depending on how big your preliminary assembly is

Blast this file against the NCBI nt database. A megablast search should be quick, and enough to estimate the approximate taxonomic composition of the preliminary assembly:

    blastn -task megablast -query $assemblyfile.r10000 -db /path/to/blastdb/nt -max_target_seqs 1 -outfmt 6 > $assemblyfile.r10000.megablast.nt

Assign taxons to the contigs at the level of genus, order, family, superfamily, and kingdom (we find that "order" works well for visualising composition, but are keeping the rest just in case):

    blast_taxonomy_report.pl \
        -b $assemblyfile.r10000.megablast.nt \
        -nodes /path/to/ncbi/taxdmp/nodes.dmp \
        -names /path/to/ncbi/taxdmp/names.dmp \
        -gi_taxid_file /path/to/ncbi/taxdmp/gi_taxid_nucl.dmp.gz \
        -t genus=1 -t order=1 -t family=1 -t superfamily=1 -t kingdom=1 \
    >$assemblyfile.r10000.megablast.nt.taxon

How to make blobology plots with R
----------------------------------

Requirements:

1. `blobology.R` from this repository
2. `$assemblyfile.r10000.megablast.nt.taxon` - a file with taxons assigned for a set of contigs from the preliminary assembly 
3. `outputprefix.lencovgc.txt` - tab delimited file with len cov gc information obtained previously using `sam_len_cov_gc_insert.pl` script above

Take the file with taxon assignments and use it to append a taxon column to the tab-delimited file with len cov gc information. Where a taxon identification is not available, use "Not annotated":

    for file in *lencovgc.txt
    do
        cat $assemblyfile.r10000.megablast.nt.taxon $file | 
        perl -anF"\t" -e '
            chomp;
            /^(\S+).*\torder\t([^\t]+)/ and $o{$1} = $2 and next;
            if ($F[2] =~ /^\d+$/ ) { print "$_\t" . (exists $o{$F[1]} ? $o{$F[1]} : "Not annotated") . "\n" }
        ' >> lencovgc.taxon
    done    

Use ggplot2 in R to make a plot showing the taxon-annotated gc-cov blob plots for each library:

    Rscript --vanilla blobology.R lencovgc.taxon 0.005 1 2
    
The three arguments to blobology.R are:

1. `lencovgc.taxon` - the tab delimited file with columns: libraryname, contigid, length, coverage, gc, taxon 
2. `0.005` - a minimum presence threshold. A taxon must be present in at least this proportion of annotated contigs before it is reported. Without this cutoff, even a dozen random hits can quickly swamp the plot legend and make it impossible to see which taxa are present.
3. `1 2` - this last pair of numbers is purely aesthetic - if there are multiple libraries being plotted, this pair specifies the number of rows and columns in which the plots are laid out. In our example, we have only two libraries, so `1 2` or `2 1` etc would have worked fine. With only 1 library, you will need `1 1`

How to make taxon-specific blast databases
------------------------------------------

Once you've made a GC-cov blob plot, you might want to make taxon specific databases to do a more sensitive search against.

For example, the contaminants in figure http://www.ncbi.nlm.nih.gov/pmc/articles/PMC3294205/figure/Fig3/ were all Proteobacteria (NCBI taxonomy ID 1224). On the NCBI website, we can easily restrict the target database to a particular taxon. However, for large-scale searches on a local computer/cluster, we will need a subset of the NCBI database specific to this taxon.

Requirements:

1. Taxonomy ID from NCBI for the subset you want to create: e.g., 1224 for Proteobacteria
2. Local copy of NCBI blast database (eg: nt or nr)
3. Install BLAST+ suite from ftp://ftp.ncbi.nih.gov/blast/executables/blast+/LATEST

        curl "http://www.ncbi.nlm.nih.gov/sviewer/viewer.fcgi?tool=portal&sendto=on&db=nuccore&dopt=gilist&qty=2000000&filter=all&term=txid1224\[Organism:exp\]" >txid1224.gids
        
        blastdb_aliastool -dbtype nucl -gilist txid1224.gids -db /path/to/blastdb/nt -out nt_Proteobacteria

Remember:

1. get the db=nuccore part right (change to db=protein for nr)
2. qty=NUM should be greater than total number of sequences

How to separate contigs based on which taxon-specific blast database they hit better
------------------------------------------------------------------------------------

Requirements:

1. Tabular blast hits to first taxon-specific blast dabatabse, e.g., `prelim_contigs_nt_Nematoda.m8.txt`
2. Tabular blast hits to second taxon-specific blast dabatabse, e.g., `prelim_contigs_nt_Proteobacteria.m8.txt`
3. `blast_separate_taxa.pl` from this repository

The following command will create three files: `prelim_contigs_nt_Nematoda.m8.txt.only`, `prelim_contigs_nt_Proteobacteria.m8.txt.only`, and `hitting.both`, and will classify query contigs from the blast results files into one of these three files.

    blast_separate_taxa.pl -d 50 \
        -b1 prelim_contigs_nt_Nematoda.m8.txt \
        -b2 prelim_contigs_nt_Proteobacteria.m8.txt

* `-d 50` will require a minimum bitscore hit of 50 for a hit to be considered. This is also the minimum difference needed for a contig to be classified as hitting one database better if the contig hits both databases. If the score or the difference is <50, then that contig will end up in `hitting.both`
* `-b1` name of file with tabular blast hits to first blast database
* `-b2` name of file with tabular blast hits to second blast database

How to extract reads (and their pairs) that map to a set of desired contigs in the preliminary assembly
------------------------------------------------------------------------------------------

Once a set of contigs has been identified for reassembly by a series of positive and negative filtering steps, we need to extract the reads that belong to those contigs.

Requirements:

1. Read-sorted BAM file with interleaved reads mapped to preliminary assembly. e.g., `$assemblyfile.$lib.bowtie2.bam` above
2. Text file with list of desired contig ids, one per line
3. `bowtie2_extract_reads_mapped_to_specific_contigs.pl` from this repository

    bowtie2_extract_reads_mapped_to_specific_contigs.pl \
        -s <(samtools view $assemblyfile.$lib.bowtie2.bam) \
        -id contigids1.txt -id contigids2.txt -u -o $assemblyfile.$lib

* `-o` optional output prefix
* `-s` specifies a SAM alignment file with interleaved reads and soft clipping (i.e the full read is present). Because we have a BAM file instead, we use `<(samtools view $assemblyfile.$lib.bowtie2.bam)`
* `-i` text file with contig ids, one per line. Can be specified multiple times to create multiple sets of reads. For example, if we have Nematoda.contigids.txt and Proteobacteria.contigids.txt, then this script will create an interleaved fasta file in gzipped format with the name `outputprefix.Nematoda.contigids.txt.mapped.reads.fa.gz` and `outputprefix.Proteobacteria.contigids.txt.mapped.reads.fa.gz`
* `-u` is optional. If provided, a file called `outputprefix.unmapped.reads.fa.gz` is created with reads that did not map to any of the contigid file sets above

This set of reads can now be reassembled stringently.

How to find mate-paired reads that don't map to the ends of contigs and use those to conservatively scaffold the contigs
------------------------------------------------------------------------------------------

Requirements:

1. `clc_ref_assemble` mapping program from the CLC Bio assembly cell suite (tested on version 4.06 beta). A bowtie2 based version will be coming soon
2. Mate-pair (MP) read library with interleaved reads in fasta format `mplib=mpreads.interleaved.fasta.gz`
3. Assembly to be scaffolded `assemblyfile=scaffolds.fa`
4. `clc_cas_to_sspace_tab.bash` from this repository
5. `fastaqual_multiline_to_singleline.pl` from this repository (used by `clc_cas_to_sspace_tab.bash`)
6. SSPACE installed from http://baseclear.com/landingpages/sspacev12/

The problem with many MP libraries is that they are heavily contaminated with short-inserts because the Illumina MP protocol is a biotin-based enrichment process. Therefore, the library is only "enriched" for long-insert MP reads. Short-insert PE reads might make up more than 50% of the library, confounding assembly programs.

We found that one of our 4 kbp MP libraries had >50% reads mapping as "innies" (FR orientation) at 200-500 bp sizes and only 10% mapping as "outies" (RF orientation) at 3-5 kbp. To use the remaining reads to scaffold the draft genome assembly, we wanted to eliminate the reads that were actually short-insert. However, if one read matches to one contig, and its pair matches to another contig, how would we know if that pair came from a short (200-500 bp) PE fragment, or a long (4 kbp) MP fragment?

Our solution for eliminating short-insert reads was:

* map the whole MP library back to the genome assembly
* discard all read pairs that mapped to the same contig (because we are only interested in those read pairs that can help us scaffold across contigs)
* discard all read pairs where one of the reads mapped within 600 bp of the end of a contig. This was the key step, because it enabled us to be sure that if the two reads in a pair mapped to two different contigs, their insert size was >600 bp (even in the worst case scenario that the ends of the two contigs overlapped)

Once we had pulled out this set of reads, we were able to use a standalone scaffolder such as SSPACE to bridge the contigs.

Steps:

Map the interleaved MP reads to the assembly

    assemblyfile=scaffolds.fa
    mplib=mpreads.interleaved.fasta.gz
    
    clc_ref_assemble -d $assemblyfile -q $mplib -o $assemblyfile.$mplib.l95s95.cas -l 0.95 -s 0.95
    
* `-d` is the reference sequence fasta file
* `-q` is the read file
* `-o` is the output filename - CLC's propriety CAS format
* `-l 0.95 -s 0.95` says that at least 95% of the read **l**ength must match with at least 95% **s**imilarity. 

Next step:

    clc_cas_to_sspace_tab.bash $assemblyfile.$mplib.l95s95.cas $assemblyfile 600
    
The three arguments to this script are:

1. The CLC mapping file in CAS format
2. The assembly fasta file (needed, so that the length of each contig can be calculated, to see if a read is mapping within a certain distance from the ends)
3. The distance from the contig ends where a MP read is not allowed to map. Default 600

The output is a file called `$assemblyfile.$mplib.l95s95.cas.dist600.tab` which is in a tab delimited format as used by SSPACE, where each line defines the contig coordinates that a pair of reads maps to:  
contig1     start1  end1    contig2     start2  end2

To use this file in SSPACE, look up the documentation for SSPACE.

How to run khmer to digitally normalise reads
---------------------------------------------

Requirements:

1. Install screed from https://github.com/ctb/screed
2. Install khmer  from https://github.com/ctb/khmer - this tutorial site might help: http://ged.msu.edu/angus/diginorm-2012/tutorial.html

Typically, you might run *khmer* with a high-pass and a low-pass filter. First remove high coverage reads that don't contribute new k-mers beyond coverage 20, and then remove low coverage reads that have infrequent k-mers (e.g, if coverage <5 is likely to be sequencing errors or non-abundant contaminants). In some cases, only the high-pass filter is used.

khmer can be run on _1 forward and _2 reverse files separately, but we recommend that the files are interleaved before running, otherwise you might end up with almost all _2 reads discarded if all k-mers are found in the _1 file.

    shuffleSequences_fastx.pl 4 <(zcat g_ju800_110714HiSeq300_1.clean.txt.gz) <(zcat g_ju800_110714HiSeq300_2.clean.txt.gz`) | gzip >g_ju800_110714HiSeq300.clean.txt.gz

In this case, I am using the scythed and sickled cleaned read fastq files generated above.

Now, run the high-pass filter first:

    normalize-by-median.py g_ju800_110714HiSeq300.clean.txt.gz -C 20 -k 25 -N 4 -x 2e8 -s interleaved.C20.kh -R interleaved.C20.report

* The command above will take the gzipped cleaned interleaved fastq file specified and normalize-by-median all reads to a max-coverage of `-C 20`
* `-k 25` specifies that k-mers of size 25 are used for normalization (this is a reasonable default)
* `-N 4 -x 2e8` specifies how many blocks are used and how big each is (this settig will take up 4 x 2e8 = 3.2 GB of memory. You might have to increase -x to 1e9 or more if khmer doesn't run on your data)
* `-s` specifies the name of the file where the k-mer counts are stored and `-R` specifies a report file

The output of this command will be a .keep file: `g_ju800_110714HiSeq300.clean.txt.gz.keep` which is a fasta file (khmer doesn't output fastq quality files).

To run the low-pass filter (in this case, remove reads with coverage less than `-C` 5X):

    filter-abund.py -C 5 interleaved.C20.kh g_ju800_110714HiSeq300.clean.txt.gz.keep

The command above takes the k-mer count file `interleaved.C20.kh` and whenever it sees reads in `g_ju800_110714HiSeq300.clean.txt.gz.keep` that have very infrequent k-mer coverage, it discards them. The final output file of this step is named `g_ju800_110714HiSeq300.clean.txt.gz.keep.abundfilt`.

One important thing to remember is that khmer works on individual reads and not on pairs. As a result, the pairing information of a read is lost if its pair is discarded. To get around this problem, we use the script `khmer_re_pair.pl` which pulls in the paired reads:

    khmer_re_pair.pl -i g_ju800_110714HiSeq300.clean.txt.gz -k g_ju800_110714HiSeq300.clean.txt.gz.keep.abundfilt > g_ju800_110714HiSeq300.clean.txt.gz.keep.abundfilt.repaired

Currently, `khmer_re_pair.pl` needs the complete file to be an interleaved fasta file (can be gzipped). For each read in the khmer-ed fasta file `g_ju800_110714HiSeq300.clean.txt.gz.keep.abundfilt`, it looks up the original interleaved fasta file and outputs both reads in the pair.

Note: all of this has to be done for the other library (600 bp) as well. i.e. we should be left with khmer-ed and repaired files:

* `g_ju800_110714HiSeq300.clean.txt.gz.keep.abundfilt.repaired`
* `g_ju800_110714HiSeq600.clean.txt.gz.keep.abundfilt.repaired`


How to assemble RNA-Seq reads using SOAPdenovo-Trans
----------------------------------------------------

Requirements:

1. Illumina Paired-End RNA-Seq reads
2. Accurate insert size estimate for PE reads
3. SOAPdenovo-Trans from http://soap.genomics.org.cn/SOAPdenovo-Trans.html - Release 1.01, 12-22-2011

Make a config file (called 'config' in this case) with the right path to your read files (in fastq format)

    max_rd_len=38
    [LIB]
    avg_ins=100
    rank=1
    q1=../s_5_AllRead1.fastq
    q2=../s_5_AllRead2.fastq

Run SOAPdenovo-Trans

    SOAPdenovo-Trans_1.01/SOAPdenovo-Trans-31kmer all -s config -M 3 -D 2 -a -p 32 -o M3D2

* all - to do all steps of the assembly (graph building, scaffolding etc)
* -s config file
* -M mergeLevel(default 1,min 0, max 3): the strength of merging similar sequences during contiging
* -D EdgeCovCutoff: delete edges with coverage no larger than (default 1)
* -a initMemoryAssumption: Initiate the memory assumption to avoid further reallocation
* -p n_cpu(default 8): number of cpu for use
* -o OutputFile: prefix of output file name

The file you want will be M3D2.scafSeq (or whatever output prefix you specify)

How to align ESTs and protein sequences to genome assemblies to assess assembly contiguity and completeness
-------------------------------------------------------------------------------------

Requirements:

1. Genome assembly nucleotide fasta files
2. EST nucleotide fasta file
3. Protein fasta file
4. BLAT
5. tblastn from NCBI's blast+ suite
6. `blastm8_filter.pl`, `bedsum`, and `seq_st_en_merge_overlapping.pl` from this repository

To compare multiple genome assemblies, keeping the same file extension (eg, .fna) makes it easier to run the comparisons in parallel.

### Proteins

First, make blast databases and run tblastn:

    proteinfile=related_species.faa
    
    parallel "makeblastdb -dbtype nucl -in {}" ::: *.fna

    parallel "tblastn -query $proteinfile -db {} -evalue 1e-5 -outfmt '6 std qlen slen' -max_target_seqs 10 -out $proteinfile.{}.tblastn -num_threads 2" ::: *.fna

To assess contiguity, count how many protein query sequences were >70% covered by just one target genome hit (and the span of amino acids in such proteins):

    parallel -k  "echo {}; \
    zcat {} | blastm8_filter.pl -b - -m 1 -c -qcov 0.7 -query $proteinfile | cut -f1 | uniq | wc -l; \
    zcat {} | blastm8_filter.pl -b - -m 1 -c -qcov 0.7 -query $proteinfile | cut -f1,7,8 | seq_st_en_merge_overlapping.pl | bedsum" ::: $proteinfile.*.fna.tblastn

`blastm8_filter.pl` uses the following options:

- `-b` input tabular blast result (tblastn in this case)
- `-m 1` Take only the top hit for each query
- `-c` Combine HSPs within a hit when calculating coverage
- `-qcov 0.7` Filter out those hits where less than 70% of the query is covered by alignments
- `-query` Name of query protein file (needed to calculate the length of each protein sequence to estimate the coverage)

`seq_st_en_merge_overlapping.pl` takes sequence names and start-stop intervals, merges overlapping intervals, and returns only non-overlapping intervals.

`bedsum` is a one line awk script for summing the total span of all sequence intervals (`awk '{span+=$3-$2+1}END{print span}'`)

To assess completeness, count how many protein query sequences (and the span of amino acids in such proteins) were >50% covered if all hits to the genome were considered:

    parallel -k  "echo {}; \
    zcat {} | blastm8_filter.pl -b - -c -qcov 0.5 -query $proteinfile | cut -f1 | uniq | wc -l; \
    zcat {} | blastm8_filter.pl -b - -c -qcov 0.5 -query $proteinfile | cut -f1,7,8 | seq_st_en_merge_overlapping.pl | bedsum" ::: $proteinfile.*.fna.tblastn

### ESTs

Run BLAT for the query EST set against multiple assemblies, but get the result in tabular blast format:

    estfile=same_species_est.fna
    parallel "blat -out=blast8 {} $estfile.{}.blat.blast8" ::: *.fna
    
The remaining steps are the same as above.
 
