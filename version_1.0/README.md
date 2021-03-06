# Manual of MitoZ

## About MitoZ
MitoZ is a Python3-based toolkit which aims to automatically filter pair-end raw data (fastq files), assemble genome, search for mitogenome sequences from the genome assembly result, annotate mitogenome (genbank file as result), and mitogenome visualization. MitoZ is available from `https://github.com/linzhi2013/MitoZ`.


## System requirment

### Platform
MitoZ is developed and tested under `Linux version 2.6.32-696.el6.x86_64 (mockbuild@c1bm.rdu2.centos.org) (gcc version 4.4.7 20120313 (Red Hat 4.4.7-18) (GCC) ) #1 SMP Tue Mar 21 19:29:05 UTC 2017`. 

### Harddisk space
\>100G

### Memory
It takes ~100G when we tested MitoZ with `--thread_number 16`.

## Get started

See `INSTALL.md` for installation instruction.

MitoZ includes multiple functions, including `all`, `all2`, `filter`, `assemble`, `findmitoscaf`, `annotate` and `visualize`.


**Important: make sure you are in the `mitozEnv` environment when you run MitoZ!**

    source activate mitozEnv

### Data requirement
#### Pair-end (PE) fastq files
e.g. `raw.1.fq.gz` and `raw.2.fq.gz` or `clean.1.fq.gz` and `clean.2.fq.gz`

The read length should be >= 71bp (PE71). Typically, I use data of PE100 or PE150 sequencing.

The length of read1 and read2 must be equal. You should trim your data (e.g. use the option `--keep_region` in `filter` function of MitoZ) before running MitoZ.

The insert size of pair-end library should be small insert size (<1000 bp). I do not recommend to use data of large insert size (>1000bp, mate-pair library), because this kind of data usually is not good as small insert size data.

About 2 to 3G base pair (bp) is enough for mitochondrial genome assembly.

#### Fasta file
When you annotate a mitogenome sequence(s) stored in fasta file, the sequence id can not be too long, or MitoZ will fail. This is for some limitation in BioPython that MitoZ invokes.

#### Genetic code
It is important to set a correct genetic code for MitoZ (`--genetic_code` option). Usually, arthropods use the invertebrate mitochondrial code (`--genetic_code 5`), and mammals use the vertebrate mitochondrial code (`--genetic_code 2`).Please refer to https://www.ncbi.nlm.nih.gov/Taxonomy/Utils/wprintgc.cgi for more details.

MitoZ use the genetic code for annotating the Protein coding genes (PCGs) of mitochondrial genome.

#### Taxa group
It is also important to set a correct taxa group for MitoZ (`--clade` option).

MitoZ use the `--clade` option to choose corresponding database (HMM modules, CM modules, protein reference sequences).

### 1 all

`all` function requires only two input pair-end fastq files, and outputs a genbank file containing mitochondrial genome sequences and annotation information.

#### 1.1 Input files
Pair-end(PE) fastq files (`raw.1.fq.gz` and `raw.2.fq.gz`), and optional files `1.adapter.list.gz` and `2.adapter.list.gz`.

#### 1.2 Example

    python3 MitoZ.py all --genetic_code 5 --clade Arthropoda --outprefix test \
    --thread_number 12 \
    --fastq1 raw.1.fq.gz \
    --fastq2 raw.2.fq.gz \
    --fastq_read_length 150 \
    --insert_size 250  \
    --run_mode 2 \
    --filter_taxa_method 1 \
    --requiring_taxa 'Arthropoda'

For more details, please refer to `python3 MitoZ.py all -h`


### 2 all2

`all2` is amolst the same as `all`, except that `all2` doesn't filter the input fastq files.

#### 2.1 Input files
Pair-end(PE) fastq files (`clean.1.fq.gz` and `clean.2.fq.gz`).

#### 2.2 Example
    
    python3 MitoZ.py all --genetic_code 5 --clade Arthropoda --outprefix test \
    --thread_number 8 \
    --fastq1 clean.1.fq.gz \
    --fastq2 clean.2.fq.gz \
    --fastq_read_length 150 \
    --insert_size 250 \
    --run_mode 2 \
    --filter_taxa_method 1 \
    --requiring_taxa 'Arthropoda'


### 3 filter

`filter` is to filter input raw fastq files (`raw.1.fq.gz` and `raw.2.fq.gz`), outputs clean fastq files (`clean.1.fq.gz` and `clean.2.fq.gz`)

#### 3.1 Input files
Pair-end(PE) fastq files (`raw.1.fq.gz` and `raw.2.fq.gz`), and optional files `1.adapter.list.gz` and `2.adapter.list.gz`.

#### 3.2 Example

    python3 MitoZ.py filter \
    --fastq1 raw.1.fq.gz \
    --fastq2 raw.2.fq.gz \
    --fastq3 clean.1.fq.gz \
    --fastq4 clean.2.fq.gz \
    --outprefix test


### 4 assemble

`assemble` is to assemble `clean.1.fq.gz` and `clean.2.fq.gz`, search for mitochondrial sequences from the assembly. Output is a mitochondrial sequence file in fasta format.

#### 4.1 Input files
Pair-end(PE) fastq files (`clean.1.fq.gz` and `clean.2.fq.gz`).

#### 4.2 Example
    
    python3 MitoZ.py assemble --genetic_code 5 --clade Arthropoda --outprefix test \
    --thread_number 8 \
    --fastq1 clean.1.fq.gz \
    --fastq2 clean.2.fq.gz \
    --fastq_read_length 150 \
    --insert_size 250 \
    --run_mode 2 \
    --filter_taxa_method 1 \
    --requiring_taxa 'Arthropoda'


### 5 findmitoscaf

`findmitoscaf` is to search for the mitochondrial sequences from fasta file which contains non-mitochondrial sequences. Output is a mitochondrial sequence file in fasta format.

#### 5.1 Input files
* `totoal_assembly.fa`
A file contains non-mitochondrial sequences.

If `totoal_assembly.fa` is generated by `SOAPdenovo-Trans-127mer`, you can specify the option `--from_soaptrans`, and in this case, `totoal_assembly.fa` is the only input file you need.

Otherwise, you still need following two files as input,

* `clean.1.fq.gz` and `clean.2.fq.gz`


#### 5.2 Example

    python3 MitoZ.py findmitoscaf --genetic_code 5 --clade Arthropoda --outprefix test \
    --thread_number 8 \
    --from_soaptrans \
    --fastafile totoal_assembly.fa

Or,

    python3 MitoZ.py findmitoscaf --genetic_code 5 --clade Arthropoda \
    --outprefix test --thread_number 8 \
    --fastq1 clean.1.fq.gz \
    --fastq2 clean.2.fq.gz \
    --fastq_read_length 150 \
    --fastafile totoal_assembly.fa


### 6 annotate

`annotate` is to annotate the input mitogenome sequence, including protein coding genes (PCGs), tRNA genes and rRNA genes. Output is a genbank file containing mitochondrial genome sequences and annotation information.

#### 6.1 Input files
e.g. `mitogenome.fa`

A fasta file containing the mitochondrial seqeunces.

#### 6.2 Example

    python3 MitoZ.py annotate --genetic_code 5 --clade Arthropoda \
    --outprefix test --thread_number 8 \
    --fastafile mitogenome.fa


If you want to see the abundance along the mitogenome sequences, you will also need to set `--fastq1` and `--fastq2`.

### 7. visualize

`visualize` module is to visualize the genbank file.

`--bgc` parameter (in the "Outfile setting" parameter section) can be an image file (e.g. the species image). In this way,
the center part of the output figure will be the species image. I will update this parameter instruction in the program in MitoZ's future version.

#### 7.1 Input files
e.g. `mitogenome.gb`

A Genbank file.

#### 7.1 Example

    python3 MitoZ.py visualize --gb mitogenome.gb

If you want to show sequencing depth along the mitogenome, you need to set `--fastq1` and `--fastq2` and `--depth`.


## Result files

The result files should be in `outprefix.result` directory.

`work*.scafSeq` is the assembly file output by SOAPdenovo-Trans.

`*_mitoscaf.fa.fsa` is the mitogenome in fasta format.

`*_mitoscaf.fa.gbf` is the mitogenome in Genbank format.

`*_mitoscaf.fa.val` and `errorsummary.val` are output by tbl2asn, you should check if there are annotation problems.

`*.statistic`: how many PCGs, rRNA genes, and tRNA genes have been annotated for each sequence.


`visualization` (and `visualization.SeqCut`) contains files of visualized mitogenome in svg and png format.


## Multi-Kmer mode
when there are missing PCGs after you run MitoZ in quick mode (`--run_mode 2`), you can try with the multi-Kmer mode (`--run_mode 3`).

You should provide the quick mode assembly as input, including files:

1. `work71.hmmout.fa`, or a file (e.g. `quickMode.fa`) which you convince containing the correct mitogenome sequences for your sample. And manually create a file (e.g. `quick_mode_fa_genes.txt`) describing what PCG genes on each sequence, format:

        seqid1 PCG1 PCG2
        seqid2 PCG3

2. `work71.hmmtblout.besthit.sim.filtered.fa`

3. `work71.hmmtblout.besthit.sim.filtered.high_abundance_*X.reformat.sorted`

in the directory of `outprefix.assembly`.


#### 7.1 Example
    
    python3 MitoZ.py all2 --genetic_code 5 --clade Arthropoda --outprefix test \
    --thread_number 12 --fastq1 clean.1.fq.gz --fastq2 clean.2.fq.gz \
    --fastq_read_length 150 --insert_size 250 \
    --run_mode 3 \
    --filter_taxa_method 1 \
    --requiring_taxa 'Arthropoda' \
    --quick_mode_seq_file quickMode.fa \
    --quick_mode_fa_genes_file quick_mode_fa_genes.txt  \
    --missing_PCGs ND4L ND6 ND2 \
    --quick_mode_score_file work71.hmmtblout.besthit.sim.filtered.high_abundance_10.0X.reformat.sorted  \
    --quick_mode_prior_seq_file work71.hmmtblout.besthit.sim.filtered.fa

The result file is `outprefix.multiKmer_seq_picked.clean.fa` under directory `outprefix.assembly2`.


## Useful scripts

    python3 bin/annotate/genbank_file_tool.py
    usage: genbank_file_tool.py [-h] {cut,comrev,sort,select} ...

    Description
        A tool to deal with genbank records.

    Version
        0.0.1

    Author
        mengguanliang(at) genomics (dot) cn, BGI-Shenzhen.

    positional arguments:
      {cut,comrev,sort,select}
        cut                 cutting sequences (5' and/or 3' end).
        comrev              get complement reverse of genbank records
        sort                sort the gene orders (input should all be circular
                            records!!!)
        select              output specific genbank records

    optional arguments:
      -h, --help            show this help message and exit

## Citation

    Guanliang Meng, Yiyuan Li, Chentao Yang, Shanlin Liu. MitoZ: A toolkit for mitochondrial genome assembly, annotation and visualization; doi: https://doi.org/10.1101/489955

