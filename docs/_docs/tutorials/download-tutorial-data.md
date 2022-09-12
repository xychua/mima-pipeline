---
title: Download tutorial data
---

# Tutorial data

In both tutorials [Data processing with Singularity](tutorial-with-singularity) and [Data processing without Singularity](tutorial-no-singularity), we will use data from the study by [*Tourlousse, et al. (2022)*](https://journals.asm.org/doi/10.1128/spectrum.01915-21), **Characterization and Demonstration of Mock Communities as Control Reagents for Accurate Human Microbiome Community Measures**, Microbiology Spectrum.

> This data set consists of two mock communities: *DNA-mock* and *Cell-mock*. The mock communities consists of bacteria that are mainly detected the human gastrointestinal tract ecosystem with a small mixture of some skin microbiota. The data was processed in three different labs: A, B and C. In the previous tutorial, , we only processed a subset of the samples (n=9). In this tutorial we will be working with the full data set which has been pre-processed using the same pipeline. In total there were 56 samples of which 4 samples fell below the abundance threshold and therefore the final taxonomy abundance table has 52 samples. We will train the random forest classifier to distinguish between the three labs.

- The raw reads are available from NCBI SRA [Project PRJNA747117](https://www.ncbi.nlm.nih.gov/bioproject/?term=PRJNA747117)
- There are 56 paired-end samples (112 fastq files)
  - As the data is very big we will work with a smaller subset to speed up processing
  
- This tutorial teaches you how to prepare the required raw files
  
{% include alert.html type="warning" title="warning" content="You will need about 40GB of disk space" %}

---

# Step 1) Download tutorial files

- Download and unpack [mima_tutorial.tar.gz]

{% include alert.html type="warning" title="note" content="In this and other tutorial, we will assume that the `mima_tutorial/` directory is located in your `HOME DIRECTORY (~)`. Change the paths accordingly." %}

```
$ wget
$ tar xf mima_tutorial.tar.gz
$ cd mima_tutorial
$ tree .
```

```
.
├── manifest.csv
├── metadata.tsv
├── pbs_header_func.cfg
├── pbs_header_qc.cfg
├── pbs_header_taxa.cfg
└── raw_data
```

**Data files**

| File | Description |
|:-----|:------------|
| SRA_files | contains the SRA identifier of the 9 samples used in the tutorials |
| manifest.csv | comma separated file of 3 columns, that lists the `sampleID, forward_filename, reverse_filename`|
| metadata.tsv | tab separated file detailing the metadata for downstream analysis. Usually the first column is the `sampleID` which is the same as the `manifest.csv` file|
| pbs_header_*.cfg | files are required in the [Data processing with Singularity](tutorial-with-singularity) tutorial (not here) |


# Step 2) Download SRA files

- We use [sratoolkit](https://www.ncbi.nlm.nih.gov/sra/docs/sradownload/) command line tool to download SRA files and unpack them using `fasterq-dump`
- If your system already has the `sratoolkit` then follow [Option A: download with `sratoolkit`](#option-a-download-with-sratoolkit)
- If not, then follow [Option B: download via MIMA](#option-b-download-via-MIMA)


## Option A: download with `sratoolkit`

- Download the SRA files using `prefetch` command

```
$ prefetch --option-file SRA_files --output-directory raw_data
```

- Below is the output, wait until all files are downloaded, there should be

```
2022-09-08T05:50:42 prefetch.3.0.0: Current preference is set to retrieve SRA Normalized Format files with full base quality scores.
2022-09-08T05:50:42 prefetch.3.0.0: 1) Downloading 'SRR17380209'...
2022-09-08T05:50:42 prefetch.3.0.0: SRA Normalized Format file is being retrieved, if this is different from your preference, it may be due to current file availability.
2022-09-08T05:50:42 prefetch.3.0.0:  Downloading via HTTPS...
...
```

- wait for download to finish
- check the downloaded files with the `tree` command

```
$ tree raw_data
```

```
raw_data/
├── SRR17380115
│   └── SRR17380115.sra
├── SRR17380118
│   └── SRR17380118.sra
├── SRR17380122
│   └── SRR17380122.sra
├── SRR17380209
│   └── SRR17380209.sra
├── SRR17380218
│   └── SRR17380218.sra
├── SRR17380222
│   └── SRR17380222.sra
├── SRR17380231
│   └── SRR17380231.sra
├── SRR17380232
│   └── SRR17380232.sra
└── SRR17380236
    └── SRR17380236.sra
```

- Extract the fastq files using the `fasterq-dump` command
- We'll also save some disk space by zipping up the fastq files

```
$ cd ~/mima_tutorial/raw_data
$ fasterq-dump --split-files */*.
$ bzip *.fastq
$ tree .
```

```
.
├── SRR17380115
│   └── SRR17380115.sra
├── SRR17380115_1.fastq.gz
├── SRR17380115_2.fastq.gz
├── SRR17380118
│   └── SRR17380118.sra
├── SRR17380118_1.fastq.gz
├── SRR17380118_2.fastq.gz
├── SRR17380122
│   └── SRR17380122.sra
├── SRR17380122_1.fastq.gz
├── SRR17380122_2.fastq.gz
├── SRR17380209
│   └── SRR17380209.sra
├── SRR17380209_1.fastq.gz
├── SRR17380209_2.fastq.gz
├── SRR17380218
│   └── SRR17380218.sra
├── SRR17380218_1.fastq.gz
├── SRR17380218_2.fastq.gz
├── SRR17380222
│   └── SRR17380222.sra
├── SRR17380222_1.fastq.gz
├── SRR17380222_2.fastq.gz
├── SRR17380231
│   └── SRR17380231.sra
├── SRR17380231_1.fastq.gz
├── SRR17380231_2.fastq.gz
├── SRR17380232
│   └── SRR17380232.sra
├── SRR17380232_1.fastq.gz
├── SRR17380232_2.fastq.gz
├── SRR17380236
│   └── SRR17380236.sra
├── SRR17380236_1.fastq.gz
└── SRR17380236_2.fastq.gz
```

---

## Option B: download via MIMA

- First run [Install MIMA pipeline](installation)
- Download the SRA files using `prefetch` command

```
$ singularity exec $SANDBOX prefetch --option-file SRA_files --output-directory raw_data
```

- Below is the output, wait until all files are downloaded, there should be

```
2022-09-08T05:50:42 prefetch.3.0.0: Current preference is set to retrieve SRA Normalized Format files with full base quality scores.
2022-09-08T05:50:42 prefetch.3.0.0: 1) Downloading 'SRR17380209'...
2022-09-08T05:50:42 prefetch.3.0.0: SRA Normalized Format file is being retrieved, if this is different from your preference, it may be due to current file availability.
2022-09-08T05:50:42 prefetch.3.0.0:  Downloading via HTTPS...
...
```

- wait for download to finish
- check the downloaded files

```
$ tree raw_data
```

```
raw_data/
├── SRR17380115
│   └── SRR17380115.sra
├── SRR17380118
│   └── SRR17380118.sra
├── SRR17380122
│   └── SRR17380122.sra
├── SRR17380209
│   └── SRR17380209.sra
├── SRR17380218
│   └── SRR17380218.sra
├── SRR17380222
│   └── SRR17380222.sra
├── SRR17380231
│   └── SRR17380231.sra
├── SRR17380232
│   └── SRR17380232.sra
└── SRR17380236
    └── SRR17380236.sra
```

- Extract the fastq files using the `fasterq-dump` command
- We'll also save some disk space by zipping up the fastq files using `bzip`

```
$ cd ~/mima_tutorial/raw_data
$ singularity exec $SANDBOX fasterq-dump --split-files */*. && bzip *.fastq
$ tree .
```

```
.
├── SRR17380115
│   └── SRR17380115.sra
├── SRR17380115_1.fastq.gz
├── SRR17380115_2.fastq.gz
├── SRR17380118
│   └── SRR17380118.sra
├── SRR17380118_1.fastq.gz
├── SRR17380118_2.fastq.gz
├── SRR17380122
│   └── SRR17380122.sra
├── SRR17380122_1.fastq.gz
├── SRR17380122_2.fastq.gz
├── SRR17380209
│   └── SRR17380209.sra
├── SRR17380209_1.fastq.gz
├── SRR17380209_2.fastq.gz
├── SRR17380218
│   └── SRR17380218.sra
├── SRR17380218_1.fastq.gz
├── SRR17380218_2.fastq.gz
├── SRR17380222
│   └── SRR17380222.sra
├── SRR17380222_1.fastq.gz
├── SRR17380222_2.fastq.gz
├── SRR17380231
│   └── SRR17380231.sra
├── SRR17380231_1.fastq.gz
├── SRR17380231_2.fastq.gz
├── SRR17380232
│   └── SRR17380232.sra
├── SRR17380232_1.fastq.gz
├── SRR17380232_2.fastq.gz
├── SRR17380236
│   └── SRR17380236.sra
├── SRR17380236_1.fastq.gz
└── SRR17380236_2.fastq.gz
```

- check that the filenames (columns 2 and 3) listed in the `manifest.csv` match with the fastq files present
- if not then update the `manifest.csv` file as required

```
$ cat ~/mima_tutorial/manifest.csv
```

```
Sample_ID,FileID_R1,FileID_R2
SRR17380209,SRR17380209.sra_1.fastq.gz,SRR17380209.sra_2.fastq.gz
SRR17380232,SRR17380232.sra_1.fastq.gz,SRR17380232.sra_2.fastq.gz
SRR17380236,SRR17380236.sra_1.fastq.gz,SRR17380236.sra_2.fastq.gz
SRR17380231,SRR17380231.sra_1.fastq.gz,SRR17380231.sra_2.fastq.gz
SRR17380218,SRR17380218.sra_1.fastq.gz,SRR17380218.sra_2.fastq.gz
SRR17380222,SRR17380222.sra_1.fastq.gz,SRR17380222.sra_2.fastq.gz
SRR17380118,SRR17380118.sra_1.fastq.gz,SRR17380118.sra_2.fastq.gz
SRR17380115,SRR17380115.sra_1.fastq.gz,SRR17380115.sra_2.fastq.gz
SRR17380122,SRR17380122.sra_1.fastq.gz,SRR17380122.sra_2.fastq.gz
```

Now you're ready to begin the tutorials:

- [Data processing with Singularity](tutorial-with-singularity) or
- [Data processing without Singularity](tutorial-no-singularity)

---

# References

- https://www.ncbi.nlm.nih.gov/sra/docs/sradownload/