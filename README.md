## UCEpipeline
find ultra-conserved elements (ungapped, 100% identity matches) from pairwise genome alignments

Requires:  
[LAST aligner](http://last.cbrc.jp/)  
[GNU Parallel](http://www.gnu.org/software/parallel/)  
[samtools](http://www.htslib.org/)  
[Seqmagick](https://github.com/fhcrc/seqmagick)  
[split_multifasta](http://iubio.bio.indiana.edu/gmod/genogrid/scripts/split_multifasta.pl)  
[blat2gff](http://iubio.bio.indiana.edu/gmod/tandy/blat2gff.pl)

#### Create LAST databases for each genome  
`lastdb -R11 dbname /path/to/genome.fa`  
OR    
`parallel "lastdb -R11 {.}.database {}" ::: *genomes.fa` 


See [lastDB](http://last.cbrc.jp/doc/lastdb.txt) man page for options regarding soft-masking and additional handling of simple repeats.

#### Split each genome into separate files for each chromosome/contig/scaffold  
`split_multifasta --input_file /path/to/genome.fa --output_dir genome_dir`

#### Pairwise genome alignments  
`ls genome_dir/ | parallel "lastal -j1 -r5 -q100 -b100 -k2 targetgenome.database genome_dir/{}" > genome1_genome2.maf`

See [lastal options](http://last.cbrc.jp/doc/lastal.txt). -j1 for gapless alignments, -r5 for normal match score, -q100 and -b100 for unfavorably high gap and mismatch scores, and -k1 to not skip any positions in sliding window comparison.  

#### Convert MAF alignments to FASTA and deduplicate (optional min-length argument to drop small sequences)

`parallel "perl maf2fasta.pl < {} | grep -v "=" > {.}.fa" ::: *.maf`

`parallel seqmagick mogrify --deduplicate-sequences --min-length 20 {} ::: *.fa`

#### Concatenate FASTA files, deduplicate again, and rename headers to incrementing number

`cat *last.fa > UCE.fa`

`seqmagick mogrify --deduplicate-sequences UCE.fa`

`awk '/^>/{print ">"++i; next}{print}' UCE.fa > UCEcands.fa`

#### Map UCE candidates back to each genome

`cat UCEcands.fa | parallel --pipe --recstart '>' lastal -T1 -u0 -r5 -q100 -b100 -k1 eachgenome.database - > genome_UCE.maf`

#### Convert alignment to PSL and then to GFF

`parallel "maf-convert psl {} > {.}.psl" ::: *_UCE.maf`

`parallel "./psl2gff.pl < {} > {.}.gff" ::: *.psl`

#### Parse GFFs for UCE sequence names

`for i in *UCE.gff; do awk -F';' '{print $2}' $i | sed 's/Target=//g' | awk '{print $1}' | sort | uniq > $i.names; done`

#### Do pairwise join on each name file to see which are common to all

`join genome1.names genome2.names > output1`  
`join output1 genome3.names > output2`  
etc. etc.

#### Extract reduced list from larger UCE candidate sequence file

`cat output3 | parallel -j 24 "{} samtools faidx UCEcands.fa >> UCEreduced.fa"`

#### Map only those elements surviving all vs. all comparison back to each genome

`cat UCEreduced.fa | parallel --pipe --recstart '>' lastal -T1 -u0 -r5 -q100 -b100 -k1 eachgenome.database - > genome_UCE.maf`

`parallel "maf-convert psl {} > {.}.psl" ::: *_UCE.maf`

`parallel "./psl2gff.pl < {} > {.}.gff" ::: *.psl`
