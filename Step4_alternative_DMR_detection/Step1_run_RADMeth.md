This is an example of running RADMeth, testing for differential methylation associated with sex (needs to be run separately for each covariate of interest).
Please refer to the MethPipe manual for a documentation of the usage of RADMeth: http://smithlabresearch.org/downloads/methpipe-manual.pdf
The time and memory requirements are much smaller than for PQLseq. For me, 100M memory and a couple hours were enough. I ran this as an array job, 56 jobs and 50000 CpG sites per job, 173 samples.

```sh
~/apps/methpipe-3.4.3/bin/radmeth regression -factor sex $DESIGN $PROP_TABLE > $OUT_FILE
```

Then combined the outputs, sorted by chromosome and location (similarily as in tep3_differential_methylation_PQLseq/Step3_Create_bed_file_each_covariate.R), excluded CpG sites for which RADMeth didn't converge (removed rows that contained -1 in column 5) and ran the p-value adjustment step:

```sh
~/apps/methpipe-3.4.3/bin/radmeth adjust -bins 1:200:1 rad_sorted_converged_sex_full.bed > rad_adjusted_sex_full.bed
```

RADMeth's own method for the detection of differentially methylated regions:

```sh
radmeth merge -p 0.01 rad_adjusted_sex_full.bed > dmrs_sex_full.bed
```
