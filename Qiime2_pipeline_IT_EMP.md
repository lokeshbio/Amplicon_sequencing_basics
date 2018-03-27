-   [QIIME2](#qiime2)
    -   [EMP format](#emp-format)
    -   [Import and demultiplex sequences](#import-and-demultiplex-sequences)
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
source activate qiime2
```

EMP format
----------

The EMP (Earth Microbiome Project) format always contains the amplicon reads in two files: "reads.fq.sh" file and the "barcodes.fq.sh" file. We can get these two files separately from a single fastq file from any Illumina or Ion-Torrent machine by using the commnad from Qiime1 "extract\_barcodes.py".

``` bash
extract_barcodes.py -f IonTorrent_Run2.fastq.corrected -l 8 -m ../Asg_090217_sam_map.txt
```

Here the `-l` represents the length of the barcode and `-m` the mapping file for the samples.

Import and demultiplex sequences
--------------------------------

First, we import the reads and barcodes into an artifact

``` bash
qiime tools import --type EMPSingleEndSequences --input-path Bar_a_seq --output-path Amp170209.qza
```

Then, we demultiplex the reads into samples using the mapping file.

``` bash
qiime demux emp-single --i-seqs Amp170209.qza --m-barcodes-file ../Asg_090217_sam_map.txt --m-barcodes-category BarcodeSequence --o-per-sample-sequences Amp170209.demux.qza
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
qiime dada2 denoise-single --i-demultiplexed-seqs Amp170209.demux.qza --p-trim-left 15 --p-trunc-len 270 --p-n-threads 5 --o-representative-sequences Amp170209.rep-seqs-dada2.qza --o-table Amp170209.table-dada2.qza
```

Note that the `--p-trim-left 15` is to remove the forward primer and `--p-trunc-len` is to remove the reverse primer and also based on how the quality of the sequences looked like from the previous step. `Amp170209.rep-seqs-dada2.qza` contains the representative sequences and `Amp170209.table-dada2.qza` contains their respective abundance.

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
