# BinSanity v0.1.3

Program implements Affinity Propagation to cluster contigs into putative genomes. BinSanity uses contig coverage as an input, while BinSanity-refine incorporates tetranucleotide frequencies, GC content, and an optional input of coverage. All relevant scripts to produce inputs for BinSanity are provided here.

## BinSanity ##
###Dependencies###
>Versions used at time of last update to script are provided in parenthesis.

* [Numpy](http://www.numpy.org/) (v1.11.1)
* [SciKit](http://scikit-learn.org/stable/install.html) (v0.17.1)
* [Biopython](http://biopython.org/wiki/Download) (v1.66)
* [BedTools](http://bedtools.readthedocs.io/en/latest/content/installation.html) (v2.17.0)

>Programs used to prepare input files for BinSanity and associated utility scripts include:

* [Bowtie2] (https://sourceforge.net/projects/bowtie-bio/) (v2.2.5)
* [Samtools] (http://www.htslib.org/) (v1.2)

>Program used by us to putatively analyze bin completion and redundancy for refinement:
* [CheckM] (https://github.com/Ecogenomics/CheckM)

###Input Files###
* Fasta file
* Combined Coverage Profile
```
contig  coverage-1 coverage-2 coverage-3
con-1   121        89         95
con-2   14         29         21
.......
```

###Script Usage's###
To generate input files for BinSanity the scripts `contig-coverage-bam.py` and `cov-combine.py` are provided:
* `contig-coverage-bam` generates a `.coverage` file that produces a tab delimited file containing average contig coverage from a `.BAM` file. In our tests we used Bowtie2 to produce a `.SAM` file.  To maintain consistency we used the `.coverage`suffix for all files output from this script.
```
contig-coverage-bam -f [fasta-file] -b [Bam-file] -o [out.coverage] 
```
The standard output:
```
 ---------------------------------------------------------
            Finding Length information for each Contig
 ---------------------------------------------------------
 Number of sequences processed for length: 2343
 
  ---------------------------------------------------------
    Extracting Coverage Information from provided BAM file
  ---------------------------------------------------------
  
  Number of records processed for coverage: 2343
  
  ---------------------------------------------------------
                    Building Coverage File
  ---------------------------------------------------------
  Final Contigs processed: 2343

```
An example of the output file is shown below:
```
$less sample-1.coverage

contig cov length
con-1  250 3049
con-2  215 1203
con-3  123 4032
con-4  110 5021
....
```

* `cov-combine.py` combines all coverage profiles provided by `contig-coverage-bam`into a single combined profile. <br />
<p>The `-c` flag is used to identify the suffix linking the coverage files produced via contig-coverage-bam. The `-o` flag is used to identify the name of the desired output file. the `-t` was added so that the user can decide what kind of transformation of the coverage data they desire (if any).<br />
* Currently the `-t` has six options:
* log --> Log transform
* None --> Raw Coverage Values
* X5 --> Multiplication by 5 
* X10 --> Multiplication by 10
* SQR --> Square root<br />
<p>We recommend using a log transformation for initial testing. Other transformations can be useful in cases where there is an extremely low range distribution of coverages and when coverage values are low
    
```
cov-combine -c [suffix-linking-coverage-files] -o [output-file] -t [Transformation]
```
Standard output will read:

```
--------------------------------------------------------------------
Getting coverage.......
        Finished combined coverage profiles in 650 seconds
____________________________________________________________________
```
* `Binsanity` clusters contigs based on the input of a combined coverage profile. Preference, damping factor, contig cut-off, convergence iterations, and maximum iterations can be optionally adjusted using `-p`, `-d`, `-x`, `-v`, and `-m` respectively.
```
Binsanity -f [directory-with-fasta-file] -l [fasta_file] -c [combined-cov]

-------------------------------------------------------
               Running Binsanity
          ---Computing Coverage Array ---
-------------------------------------------------------
Preference: -3
Maximum iterations: 4000
Convergence Iterations: 400
Conitg Cut-Off: 1000
Damping Factor: 0.95
.......
```
* `Binsanity-refine` is used to refine bins with high redundancy or low completion by reclustering only those contigs from bins indicated and incorporation of tetranucleotide frequenices and GC-content. K-mer, preference, damping factor, contig cut-off, convergence iterations, and maximum iterations can be optionally adjusted using `-kmer`, `-p`, `-d`, `-x`, `-v`, and `-m` respectively.

```
Binsanity-refine -f [directory-with-fasta-file] -l [fasta-file-identifier] -c [combined_cov_file]
```

##Example Problem##
>The Infant Gut Metagenome collected and curated by [Sharon et al. (2013)](http://dx.doi.org/10.1101/gr.142315.112) was clustered by us to test BinSanity. To confirm you have BinSanity working we have provided a folder `Example` containing the fasta file (`INFANT-GUT-ASSEMBLY.fa`) containing contigs for the Infant Gut Metagenome provided by [Eren et al. (2015)](https://doi.org/10.7717/peerj.1319). All files associated with our BinSanity run are also provided, which includes the combined coverage profile (produced using Bowtie2 v2.2.5 on defaults, `contig-coverage-bam.py`, and `cov-combine.py`.

To run the test use the following command while in the `Example` directory:

```
Binsanity -f . -l .fa -p -10
```
The output should be as follows:
```

        -------------------------------------------------------
                         Running Bin-sanity
                    
                    ---Computing Coverage Array ---
        -------------------------------------------------------
        
Preference: -10.0
Maximum Iterations: 4000
Convergence Iterations: 400
Contig Cut-Off: 1000
Damping Factor: 0.95

(4189, 11)
        
        -------------------------------------------------------
                    ---Clustering Contigs---
        -------------------------------------------------------
Cluster 0: 5
Cluster 1: 14
Cluster 2: 75
Cluster 3: 105
Cluster 4: 54
Cluster 5: 20
Cluster 6: 34
Cluster 7: 43
Cluster 8: 27
Cluster 9: 105
Cluster 10: 35
Cluster 11: 10
Cluster 12: 39
Cluster 13: 30
Cluster 14: 727
Cluster 15: 256
Cluster 16: 574
Cluster 17: 7
Cluster 18: 620
Cluster 19: 508
Cluster 20: 350
Cluster 21: 551

	  	Total Number of Bins: 22

        --------------------------------------------------------
              --- Putative Bins Computed in 233.362998962 seconds ---
        --------------------------------------------------------
```
##Other Useful Utilities##
###Using CheckM for a quick look###
For the purposes of our analysis we used CheckM as a means of generally indicating high and low redundancy bins to use the refinement script on. To speed up this process a script was written `checkm_analysis` to parse the output of checkM qa and separate Binsanity produced bins into categories of high redundancy, low completion, high completion, and strain redundacy.<p>

Currently the thresholds written into the script place bins into categories using the following parameters:<p>
* High completion: > 80% complete with < 10% redundancy, or > 50% with < 5% redundacy
* Low completion: < 50% complete with < 5% redundancy
* Strain Variation: >50% complete with >90% strain heterogeneity
* High Redundancy: 80% complete with >10% redundacy, or 50% complete > 5% redundacy
<p>
The program is called as follows:
`checkm_analysis -checkM [checkm_qa tab delimited output]`
<p>
It should be noted that selection of the high and low redundancy values are an arbitrary cut off and the values of generally accepted redundancy, completion, and strain heterogeneity are up for debate so it is recommended that if you use the script that you decide what the best cut off values are for your purposes.<p>
CheckM is also only one means of evaluating bins and for the best results we advocate using multiple evlaution methods before considering a bin 'High Quality'. This script is provided as a means to make refinement using BinSanity slightly simpler by quickly moving bins produced during a first pass of BinSanity into smaller categories for further analysis (Note this isn't really necessar if you have a small enough data, but for example in cases where we have produced 100's of bins using BinSanity it becomes increasingly more time consuming to manually separate the high and low redundancy bins.)


##Issues##

If an issue arises in the process of utilizing BinSanity please create an issue and we will address is as soon as possible. To expedite a response please provide any associated error messages. 
As this project is actively being improved any comments or suggestions are welcome.

##Citation##
Graham, E., Heidelberg, J. & Tully, B. BinSanity: Unsupervised Clustering of Environmental Microbial Assemblies Using Coverage and Affinity Propagation. bioRxiv, doi:10.1101/069567 (2016).


