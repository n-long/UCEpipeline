# UCEpipeline
find ultra-conserved elements (ungapped, 100% identity matches) from pairwise genome alignments

Requires:  
[LAST aligner](http://last.cbrc.jp/)  
[GNU Parallel](http://www.gnu.org/software/parallel/)

Optional (to run as shown):  
[Seqmagick](https://github.com/fhcrc/seqmagick)  
[blat2gff](http://iubio.bio.indiana.edu/gmod/tandy/blat2gff.pl)


### Create LAST databases for each genome  
`lastdb -R11 dbname /path/to/genome.fa`

