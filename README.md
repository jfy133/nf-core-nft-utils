# nft-utils

nf-test utility functions

This repository contains utility functions for nf-test.

These functions are used to help capture level tests using nf-test.

## `removeNextflowVersion()`

nf-core pipelines create a yml file listing all the versions of the software used in the pipeline.

Here is an example of this file coming from the rnaseq pipeline.

```yaml
BBMAP_BBSPLIT:
  bbmap: 39.01
CAT_FASTQ:
  cat: 8.3
CUSTOM_CATADDITIONALFASTA:
  python: 3.9.5
CUSTOM_GETCHROMSIZES:
  getchromsizes: 1.2
FASTQC:
  fastqc: 0.12.1
GTF2BED:
  perl: 5.26.2
GTF_FILTER:
  python: 3.9.5
GUNZIP_ADDITIONAL_FASTA:
  gunzip: 1.1
GUNZIP_GTF:
  gunzip: 1.1
STAR_GENOMEGENERATE:
  star: 2.7.10a
  samtools: 1.18
  gawk: 5.1.0
TRIMGALORE:
  trimgalore: 0.6.7
  cutadapt: 3.4
UNTAR_SALMON_INDEX:
  untar: 1.34
Workflow:
    nf-core/rnaseq: v3.16.0dev
    Nextflow: 24.04.4
```

This function remove the Nextflow version from this yml file, as it is not relevant for the snapshot.

Usage:

```groovy
assert snapshot(removeNextflowVersion("$outputDir/pipeline_info/nf_core_rnaseq_software_mqc_versions.yml")).match()
```

Only argument is path to the file.

## `getAllFilesFromDir()`

Files produced by a pipeline can be compared to a snapshot.
This function produces a list of all the content of a directory, and can exclude some files based on a glob pattern.
From there one can get just filenames and snapshot only the names when content is not stable.
Or snapshot the whole list of files that have stable content.

In this example, these are the files produced by a pipeline:

```bash
results/
├── pipeline_info
│   └── execution_trace_2024-09-30_13-10-16.txt
└── stable
    ├── stable_content.txt
    └── stable_name.txt

2 directories, 3 files
```

In this example, 1 file is stable with stable content (`stable_content.txt`), and 1 file is stable with a stable name (`stable_name.txt`).
The last file has no stable content (`execution_trace_2024-09-30_13-10-16.txt`) as its name is based on the date and time of the pipeline execution.

For this example, we want to snapshot the files that have stable content, and the filenames that have stable names.


```groovy
// Use getAllFilesFromDir() to get a list of all files and folders from the output directory, minus non stable files
def stable_name    = getAllFilesFromDir(params.outdir, true, ['**/execution_trace*.txt'] )
// Use getAllFilesFromDir() to get a list of all files from the output directory, minus the non stable files
def stable_content = getAllFilesFromDir(params.outdir, false, ['**/execution_trace*.txt', '**/stable_name.txt'] )
assert snapshot(
  // Only snapshot name as content is not stable
  stable_name*.name,
  // Snapshot content
  stable_content
).match()
```

First argument is the pipeline `outdir` directory path, second is a boolean to include folders, and the third is a list of glob patterns to ignore.
