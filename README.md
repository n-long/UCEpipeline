# UCEpipeline
find ultra-conserved elements (ungapped, 100% identity matches) from pairwise genome alignments

Requires:  
[LAST aligner](http://last.cbrc.jp/)  
[GNU Parallel](http://www.gnu.org/software/parallel/)  
[split_multifasta](http://iubio.bio.indiana.edu/gmod/genogrid/scripts/split_multifasta.pl)  
[Seqmagick](https://github.com/fhcrc/seqmagick)  
[blat2gff](http://iubio.bio.indiana.edu/gmod/tandy/blat2gff.pl)



### Create LAST databases for each genome  
`lastdb -R11 dbname /path/to/genome.fa`  
OR    
`parallel "lastdb -R11 {.}.database" ::: *genomes.fa`  
See [lastDB](http://last.cbrc.jp/doc/lastdb.txt) man page for options regarding soft-masking and additional handling of simple repeats.

### Split each genome into separate files for each chromosome/contig/scaffold  
`split_multifasta --input_file /path/to/genome.fa --output_dir genome_dir`

### Pairwise genome alignments  
`ls genome_dir/ | parallel "lastal -j1 -r5 -q100 -b100 -k2 tcas tmad/{} | last-split -m1 | last-postmask" > tcas_tmad.maf  
See [lastal options](http://last.cbrc.jp/doc/lastal.txt). -j1 for gapless alignments, -r5 for normal match score, -q100 and -b100 for unfavorably high gap and mismatch scores, and -k1 to not skip any positions in sliding window comparison.  
last-split takes multiple best hits across a genome and returns best match. -m1 ensures each query base pair is aligned to at most one target base pair.  
last-postmask will discard alignments if it contains mostly repeats. I have included this for clarity since I do not use it when examining conservation of repeats. 

