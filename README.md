# wgs2tree
This is a nextflow implementation of a workflow for the rapid generation of consensus phylogenomic trees from whole genome sequence data.

The workflow is as follows:

Inputs are whole genome assemblies, each sample can include multiple assemblies (ie, hap1 and 2)

1. For each assembly, single-copy orthologues are identified with Compleasm or Busco
2. For samples consisting of more than one assembly, the best non-redundant set of Busco orthologues is retrieved with Buscomp
3. Samples are grouped by BUSCO gene
4. For each BUSCO gene, a multiple sequence alignment is generated with MAFFT
5. A phylogenetic tree is generated for each alignment with IQTree2
6. A consensus phylogenomic tree is generated by coelescense with ASTRAL, or by concatenation from an alignment supermatrix with IQTree2
7. Gene concordance factors are calculated for the consensus tree with IQTree2

The output is a tree file, representing the consensus phylogenomic supertree.

## Usage

**Commmand line options**  
```
nextflow run main.nf [options]

### ~ Input/Output options ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ ###
--inPath        : Directory of input files (see below for formatting)
--outPath       : Directory of output files for all tasks, aswell as final supertree

### ~ Workflow options ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ ###
--withBusco     : When set to true, Busco will be used instead of compleasm [default=false]
--withConcat    : When set to true, the final tree will be generated by concatenation, rather than coalescense [default=false]
--geneThres     : Proportion of samples that must share any given gene for it to be used in alignment [default=1]
--lineage       : This will be used in any busco/compleasm runs performed by the workflow.
--buscoDir      : Absolute path to download/store lineages for busco.
--compleasmDir  : Absolute path to download/store lineages for Compleasm.

### ~ Nextflow options ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ ###
-c              : Path to custom config file [default=nextflow.config]
-resume         : Will resume workflow if previous run exists, using cached data
```

## Input format
The input directory should contain fasta files of assemblies and optionally busco/compleasm runs associated with these assemblies.

**One assembly per sample**  
When one assembly per sample is used, there are two options. The simplest is to just have a directory of fasta files. The labels on the final tree will be the base-name of each fasta file, e.g.  

```
inPath
├── cat.fasta
├── dog.fasta
└── mouse.fasta
```
  
You can also put them into their own subdirectories, where the name of the subdirectory will be the label on the tree, e.g.

```
inPath
├── cat
│   └── cat.fasta
├── dog
│   └── dog.fasta
└── mouse
    └── mouse.fasta
```

**Multiple assemblies per sample**  
When multiple assemblies per sample are used, these should be put into a subdirectory. The name of the subdirectory will be used as the label for the sample in the final tree, e.g.

```
inPath
├── cat
│   ├── cat_hap1.fasta
│   └── cat_hap2.fasta
├── dog.fasta
└── mouse.fasta
```

**Including busco/compleasm as inputs**  
To include busco/compleasm runs in the input, the run directory should be in the same place as it's corresponding assembly. In order for pairing of assembly and busco inputs, the busco run directory must have the same base-name as the corresponding assembly fasta file, but with a 'run_' prefix, e.g.

```
inPath
├── cat
│   ├── cat_hap1.fasta
│   ├── run_cat_hap1
│   │   └── ...
│   ├── cat_hap2.fasta
│   └── run_cat_hap2
│       └── ...
├── dog.fasta
├── mouse.fasta
└── run_mouse
    └── ...
```

Ensure that any subdirectories that are not busco runs do not contain a "run_" prefix!

## Walk-through
1. Ensure that Nextflow and Singularity are installed.

2. Download wgs2tree
```
git clone https://github.com/Pav-mis/wgs2tree
```

3. Establish input directory structure (see above).

4. run pipeline, e.g.
```
nextflow run main.nf --inPath 'assemblies' --outPath 'NF' --withBusco true --lineage 'vertebrata_odb10' --geneThres 0.9
```
OR  
  
Create a custom config file with all of your parameters and use this, e.g.
```
nextflow run main.nf -c custom.config
```
If running on slurm, wrap run command in an sbatch script, allocating a single core (see `wgs2tree.sh`). If running for the first time, allocate more memory, as initial pulling/building of singularity containers is memory intensive. 

