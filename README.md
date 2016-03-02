# UCEpipeline
find ultra-conserved elements (ungapped, 100% identity matches) from pairwise genome alignments

Requires:  
[LAST aligner](http://last.cbrc.jp/)  
[GNU Parallel](http://www.gnu.org/software/parallel/)  
[split_multifasta](http://iubio.bio.indiana.edu/gmod/genogrid/scripts/split_multifasta.pl)  
[Seqmagick](https://github.com/fhcrc/seqmagick)  
[blat2gff](http://iubio.bio.indiana.edu/gmod/tandy/blat2gff.pl)



### Create LAST databases for each genome  
`lastdb -R11 dbname /path/to/genome.fa` OR  
`parallel "lastdb -R11 {.}.database" ::: *genomes.fa`
See [lastDB](http://last.cbrc.jp/doc/lastdb.txt) man page for options regarding soft-masking and additional handling of simple repeats.

### Split each genome into separate files for each chromosome/contig/scaffold  
`split_multifasta --input_file /path/to/genome.fa --output_dir agla_busco/

