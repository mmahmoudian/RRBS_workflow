SNPs were filtered using BS-SNPer from https://github.com/hellbelly/BS-Snper, Gao Shengjie, Zou Dan, Mao Likai, et al. BS-SNPer: SNP calling in bisulfite-seq data[J]. Bioinformatics, 2015, 31(24): 4006-4008.
BS-SNPer detects SNPs from bam files (bisulfite sequencing). Also a genome file is needed (`hg19.fa` in this example).

If you have several samples from each individual, combine them, e.g. `samtools merge -cp Case1_merged.bam Case1_sample1.bam Case1_sample2.bam` (In this project we just had one per individual).

Before running BS-SNPer, you need to remove lambda phage reads from the bam files in case the alignment was done simultaneously on lambda and the reference genome.

Ran this as an array job. In this project one day was enough (for bam files of size 1-3 GB) but in other projects I've needed two weeks. Depends on the size of the bam files.

```sh
perl BS-Snper.pl --fa "${REFERENCE}" \
                 --input "${INPUTFOLDER}${FILE}.bam" \
                 --output "${OUTPUTFOLDER}snp_${FILE}_temp_file" \
                 --methcg "${OUTPUTFOLDER}meth_${FILE}_cg_temp" \
                 --methchg "${OUTPUTFOLDER}meth_${FILE}_chg_temp" \
                 --methchh "${OUTPUTFOLDER}meth_${FILE}_chh_temp" \
                 --minhetfreq 0.1 \
                 --minhomfreq 0.85 \
                 --minquali 15 \
                 --mincover 10 \
                 --maxcover 1000 \
                 --minread2 2 \
                 --errorate 0.02 \
                 --mapvalue 20 \
                 > "${OUTPUTFOLDER}SNP_${FILE}.out" \
                 2> ERR.log
```

The `ERR.log` was a huge file. I only saved the first and last 50 rows for each sample:

```sh
tail -n 50 ERR.log > "${OUTPUTFOLDER}ERR_tail_${FILE}.log"
head -n 50 ERR.log > "${OUTPUTFOLDER}ERR_head_${FILE}.log"
rm ERR.log
```

