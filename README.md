# Ampliq
Codes related to amplicon analysis.

Below are the explanations of what each of the file is about!

### Qiime2

The two files basically are the same when you want to run the Qiime2 analysis pipeline for any kind of amplicon sequencing data. Here in this code we mostly only look at the most used 16S rRNA based community analysis. The two files differ only in terms of how you import your sequencing data. Because one file has infor about single-end data from Ion-Torrent sequencing and the other has info about importing paired-end data from Illumina sequencing.

The analysis goes all the way from sequencing data to getting taxonomy and ASV abundance table.

### DADA2

This file has information about how we could just use 'DADA2' in R instead of using it inside Qiime2. It could be the case that the Qiime2 is slow sometimes in going from one step to the other. This is quicker interms of time.


```bash
fortune|cowsay -f elephant
 _________________________________________
/ I do desire we may be better strangers. \
|                                         |
| -- William Shakespeare, "As You Like    |
\ It"                                     /
 -----------------------------------------
 \     /\  ___  /\
  \   // \/   \/ \\
     ((    O O    ))
      \\ /     \ //
       \/  | |  \/ 
        |  | |  |  
        |  | |  |  
        |   o   |  
        | |   | |  
        |m|   |m|  

```
