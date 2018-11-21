-   [QIIME2](#qiime2)
    -   [Import demultiplex and trim sequences](#Import-demultiplex-and-trim-sequences)
    -   [Summarizing samples](#summarizing-samples)
    -   [DADA2 analysis](#dada2-analysis)
    -   [Visualizing files](#visualizing-files)
    -   [Merging different runs](#merging-different-runs)
    -   [Taxonomic classifications](#taxonomic-classifications)
    -   [Bar plots](#bar-plots)
    -   [Exporting classifications and table](#exporting-classifications-and-table)
    -   [Krona plots](#krona-plots)

QIIME2
======

This is the readme file for the Qiime2 pipeline to use for the analysis of 16S amplicons for example! You can activate Qiime2 by the following command:

``` bash
source activate qiime2-2018.2
```

Import demultiplex and trim sequences
---------------------------------

This step only imports Forwrd reads `forward.fastq.gz`, Reverse reads `reverse.fastq.gz` and the Barcodes `barcodes.fastq.gz` file in the EMP format

``` bash
qiime tools import --type EMPPairedEndSequences --input-path . --output-path amp_nov_2018.qza
```

Now comes demultiplexing! In here demultiplex samples based on barcodes first before you do primer/adaptor removal. In the map file you can specify a selected number of samples to make a small file. 

``` bash
qiime demux emp-paired --m-barcodes-file DG280F_map.tsv --m-barcodes-column BarcodeSequence --i-seqs ../amp_nov_2018.qza --o-per-sample-sequences DG280F.qza
```

It's time to cutadapt trimming, as it is implemented in qiime2

``` bash
qiime cutadapt trim-paired --i-demultiplexed-sequences DG280F.qza --p-front-f AATCGNTANGGGCCGTGA --p-adapter-f AGATCGGAAGAGCACACGTC --p-front-r GACCACTTGAAGAGCTGGT --p-adapter-r AGATCGGAAGAGCGTCGTGT --o-trimmed-sequences DG280F_trimmed.qza --output-dir cutadapt_di --verbose > cutadapt_trim.log
```

Summarizing samples
-------------------

Here, we summarize the demultiplexed samples and their individual sequence counts and also to look at the sequence quality plot to choose parameters for trimming in the following steps.

``` bash
qiime demux summarize --i-data Amp170209.demux.qza --o-visualization Amp170209.demux.qzv
```

Then, we can visialize the summary file in a web browser by:

``` bash
qiime tools view Amp170209.demux.qzv
```

DADA2 analysis
--------------

After the demultiplexing step, the quality control, 'ASV' selection, chimera checking and many other qulaity steps are dobe by the pipleline `DADA2` along with extracting the representative sequnces and building a table for the representatives. All of this is done by just one command:

``` bash
qiime dada2 denoise-paired --i-demultiplexed-seqs DG280F_trimmed.qza --p-trim-left-f 20 --p-trim-left-r 20 --p-trunc-len-f 200 --p-trunc-len-r 200 --o-table DG280F_table.qza --o-representative-sequences DG280F_rep-seqs.qza --p-n-threads 6
```

Note that the `--p-trim-left 10` is to remove the forward primer and `--p-trunc-len` is to remove the reverse primer and also based on how the quality of the sequences looked like from the previous step. `Amp170209.rep-seqs-dada2.qza` contains the representative sequences and `Amp170209.table-dada2.qza` contains their respective abundance.

Visualizing files
-----------------

We can visialize the different characteristics of the reprentstavive sequences and the table by the following commands:

``` bash
qiime feature-table tabulate-seqs --i-data Amp170209.rep-seqs-dada2.qza --o-visualization Amp170209.rep-seqs-dada2.qzv

qiime feature-table summarize --i-table Amp170209.table-dada2.qza --o-visualization Amp170209.table-dada2.qzv --m-sample-metadata-file Comb_4ITruns_map.txt

qiime tools view Amp170209.table-dada2.qzv
qiime tools view Amp170209.rep-seqs-dada2.qzv
```

Merging different runs
----------------------

In case there are many sequencing runs using the same barcodes for different samples, the tables and the representative sequences can be mereged together.

Note: `The parameters used above should be the same for all the runs`

``` bash
qiime feature-table merge-seq-data --i-data1 Amp161122.rep-seqs-dada2.qza --i-data2 Amp170531.rep-seqs-dada2.qza --o-merged-data merged_seq.qza

qiime feature-table merge --i-table1 Amp170531.table-dada2.qza --i-table2 Amp170602.table-dada2.qza --o-merged-table merged_table2.qza
```

Note: it is also to be noted that only two features can be merged togther once!

Taxonomic classifications
-------------------------

Now, we can classify the representative sequences to their respective taxonomic unit using the already existing reference sequences such as the SILVA database and CREST and so on.

``` bash
qiime feature-classifier classify-consensus-vsearch --i-query All_Asg_16S_rep_seqs.qza --i-reference-reads ~/Files/Database/Qiime2/Silva_128_Q2/16S_SILVA128_99_otus.qza --i-reference-taxonomy ~/Files/Database/Qiime2/Silva_128_Q2/16S_SILVA128_99_taxa.qza --p-threads 5 --output-dir Silva_classified

qiime feature-classifier classify-consensus-vsearch --i-query All_Asg_16S_rep_seqs.qza --i-reference-reads ~/Files/Database/Qiime2/Crest_Q2/Crest_97_otus.qza --i-reference-taxonomy ~/Files/Database/Qiime2/Crest_Q2/Crest_97_taxa.qza --p-threads 5 --output-dir Crest_classified
```

Bar plots
---------

The Qiime barplots can be plotted using the following commnad:

``` bash
qiime taxa barplot --i-table ../All_Asg_16S_table.qza --i-taxonomy classification.qza --m-metadata-file ../Comb_4ITruns_map.txt --o-visualization silva_taxa-bar-plots.qzv
```

Exporting classifications and table
-----------------------------------

``` bash
qiime tools export classification.qza --output-dir .
qiime tools export All_Asg_16S_table.qza --output-dir .
```

The classfication table has a name `taxonomy.tsv` as an output with taxonomy for each ASV and the table out put is in biom format!

Krona plots
-----------

Here, we would combine the taxonomy classifications and the abundance table from the biom table to make Krona plots

``` bash
biom convert -i All_Asg-table.biom -o All_Asg_asv_table.tab --to-tsv
krona_qiime.py ../taxonomy.tsv ../../All_Asg_asv_table.tab
```

The above `biom` command will create a normal ASV abundance table. then, we combine the the taxonomic classification of the ASV to their abundance using my own `krona_qiime.py` command. This will create text files for each sample in the analysis. then we would combine the text files to make the krona plots.

``` bash
ktImportText text_files/Str* text_files/Sec* -o Piran_silva.html
```
