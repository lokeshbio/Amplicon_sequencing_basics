-   [Taxa](#just-taxa)
-   [DADA2](#dada2)


Just Taxa
---------

If you just want to assign taxonomy using DADA2 package, as it is a bit faster than vsearch, use the following code.

``` r
library(dada2); packageVersion("dada2")
seqs <- getSequences("OTUs_frm_MadsAlb.fa",collapse = FALSE)
taxa80 <- assignTaxonomy(seqs, "Dada2_silva_nr_v132_train_set.fa.gz", minBoot=80, multithread=10)
write.table(taxa, "OTUs_frm_MadsAlb_Dada2_taxonomy.tab", sep="\t")
```



DADA2
-----

Here is the pipeline for analysing the 16S data just using the dada2 pipeline in R without the need for Qiime2!! Using Qiime2 is still better since we can combine this methods like UniFrac and so on!! But this is just the outline of how we could use it, in case the Qiime2 isn't working well for the case for some reason.

``` r
library(dada2); packageVersion("dada2")
seq_path <- "~/Files/Loki_16S/All_16S_MISEQ_27_03_2018/Piran_Analysis/Adapter_clipped_frm_LGC/Piran_analysis/Samples/"
list.files(seq_path)
fnFs <- sort(list.files(seq_path, pattern="_R1_clipped.fastq", full.names = TRUE))
fnRs <- sort(list.files(seq_path, pattern="_R2_clipped.fastq", full.names = TRUE))
# Extract sample names, assuming filenames have format: SAMPLENAME_XXX.fastq
sample.names <- sapply(strsplit(basename(fnFs), "_"), `[`, 1)
plotQualityProfile(fnFs[1:2])
plotQualityProfile(fnRs[1:2])
filt_path <- file.path(seq_path, "filtered") # Place filtered files in filtered/ subdirectory
filtFs <- file.path(filt_path, paste0(sample.names, "_F_filt.fastq.gz"))
filtRs <- file.path(filt_path, paste0(sample.names, "_R_filt.fastq.gz"))
out <- filterAndTrim(fnFs, filtFs, fnRs, filtRs, truncLen=c(180,180), trimLeft=c(10,10),
                     maxN=0, maxEE=c(2,2), truncQ=2, rm.phix=TRUE,
                     compress=TRUE, multithread=TRUE) # On Windows set multithread=FALSE
head(out)
errF <- learnErrors(filtFs, multithread=TRUE)
errR <- learnErrors(filtRs, multithread=TRUE)
plotErrors(errF, nominalQ=TRUE)
plotErrors(errR, nominalQ=TRUE)
derepFs <- derepFastq(filtFs, verbose=TRUE)
derepRs <- derepFastq(filtRs, verbose=TRUE)
# Name the derep-class objects by the sample names
names(derepFs) <- sample.names
names(derepRs) <- sample.names
dadaFs <- dada(derepFs, err=errF, multithread=TRUE)
dadaRs <- dada(derepRs, err=errR, multithread=TRUE)
mergers <- mergePairs(dadaFs, derepFs, dadaRs, derepRs, verbose=TRUE)
head(mergers[[1]])
seqtab <- makeSequenceTable(mergers)
dim(seqtab)
table(nchar(getSequences(seqtab)))
seqtab.nochim <- removeBimeraDenovo(seqtab, method="consensus", multithread=TRUE, verbose=TRUE)
sum(seqtab.nochim)/sum(seqtab)
table(nchar(getSequences(seqtab.nochim)))
getN <- function(x) sum(getUniques(x))
track <- cbind(out, sapply(dadaFs, getN), sapply(mergers, getN), rowSums(seqtab), rowSums(seqtab.nochim))
colnames(track) <- c("input", "filtered", "denoised", "merged", "tabled", "nonchim")
rownames(track) <- sample.names
dim(track)
track
taxa <- assignTaxonomy(seqtab.nochim, "~/Files/Database/SILVA_132/Dada2_silva_nr_v132_train_set.fa.gz", multithread=TRUE)
temp <- t(seqtab.nochim)
write.table(temp, "Piran_adap_Dada2_abund_table.tab", sep="\t")
write.table(taxa, "Piran_adap_Dada2_taxonomy.tab", sep="\t")
```
