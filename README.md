# Genome Annotation
[![hackmd-github-sync-badge](https://hackmd.io/jqXB_tEgQtWxRVZc1ChNeQ/badge)](https://hackmd.io/jqXB_tEgQtWxRVZc1ChNeQ)

###### tags: `genome annotation` `singularity` `BRAKER3`

###### Last updated: Dec 1, 2023
------------------------------------------------------------------------

This tutorial was written for students of the Genomes and Genome Evolution course at Texas Tech and tested at the [High Performance Computing Cluster](https://https://www.depts.ttu.edu/hpcc/) (HPCC) of this University.

## <i class="fa fa-download fa-lg"></i> Pre-requisites

You will only need [Singularity](https://apptainer.org/) installed in your computer or cluster. Singularity, now also called Apptainer, is a tool that enables you to create and run applications in containers. What are containers, you may ask? Of course the name gives away their function—they contain things. In this case, they encapsulate all the software and dependencies you need to run a specific application. This is **AWESOME** because you only need to have a **S**ingularity **I**mage **F**ormat (SIF) and nothing else.

Essentially a SIF file is a pre-made, tiny operating system that contains everything you need to run a process. Once that image is created, you can use it instead of installing every piece of software yourself. This is terrific because otherwise it may take you months to install everything you need. I know this from experience. Take a look at [this extensive list](https://github.com/Gaius-Augustus/BRAKER#braker) of software, dependencies, and setup steps you would have to perform first before attempting to use BRAKER3 to annotate a genome.

 Luckily for us, Singularity is already installed on the HPCC. If you are using your own computer, you can follow the instructions [here](https://docs.sylabs.io/guides/3.0/user-guide/installation.html) to install it.

Lastly, the files you will use in this tutorial are available at my [GitHub repository](https://github.com/Xacfran/Genome-Annotation-with-Braker3) if you are not a Tech student. If you are working on the HPCC, you can copy them to your own directory with:

```bash
cp /lustre/work/frcastel/software/braker/scaffold_myo_myo.fasta .
cp /lustre/work/frcastel/software/braker/filtered_mammalia.fasta .
```

The first file is the largest scaffold of the genome assembly of the bat *Myotis myotis* you worked with some weeks ago, and the second one is a filtered version comprising 3,835,124 proteins from the largest database of the [Vertebrata dataset](https://bioinf.uni-greifswald.de/bioinf/partitioned_odb11/) from [OrthoDB](https://www.orthodb.org/), which houses over 9 million proteins. We'll use this dataset to train the gene models, but first lets get the file that will allow us to use BRAKER3.

## <i class="fa fa-gears fa-lg"></i> Installation

To get the BRAKER3 SIF file you simply need to run:

```bash
singularity build braker3.sif docker://teambraker/braker3:latest
```

## <i class="fa fa-code fa-lg"></i> Usage

BRAKER3 runs in different modes. It can make use of the genome alone, genome + proteins, genome + RNA-Seq, or all of them. You can find the details of how each mode works [here](https://github.com/Gaius-Augustus/BRAKER#overview-of-modes-for-running-braker).
In this tutorial we will use proteins, but first lets use some test scripts to make sure it works.

### Test data

We will download 3 test scripts that will run BRAKER3 in different modes.

```bash
singularity exec -B $PWD:$PWD braker3.sif cp /opt/BRAKER/example/singularity-tests/test1.sh .
singularity exec -B $PWD:$PWD braker3.sif cp /opt/BRAKER/example/singularity-tests/test2.sh .
singularity exec -B $PWD:$PWD braker3.sif cp /opt/BRAKER/example/singularity-tests/test3.sh .
```

 For this tutorial we will only use the first one `test1.sh`, but you can try the others on your own. Before running the test, let's make sure we incorporate BRAKER3 into our path with `export BRAKER_SIF=/path/to/your/braker3.sif`, and then run the script with `bash test1.sh`.

After a few minutes you will notice a directory named `test1` containing the annotation results. You can take a look at the files with `ls test1`. If you see the files: `braker.gtf`, `braker.aa` and `braker.codingseq`, and they are not empty, then everything worked fine. Let's proceed with our own data.

### Annotating a bat genome assembly

As discussed in class, annotating a genome is a complex process that invokes several pieces of software, each one of them with specifical tasks. BRAKER3 is a pipeline that automates this process so the code looks as simple as any other you have used during the semester as you can see below.

```bash
#!/bin/bash
#SBATCH --job-name=myo_braker
#SBATCH --output=%x.%j.out
#SBATCH --error=%x.%j.err
#SBATCH --partition=nocona
#SBATCH --nodes=1
#SBATCH --ntasks=36
#SBATCH --mem=0

# Get the SIF file into the path
export BRAKER_SIF=/path/to/your/braker3.sif

# Create the working directory or delete it if it exists
# You can change the name myo_myo to anyting you think is appropriate
WORKDIR=/path/to/your/working/directory
WD=myo_myo


if [ -d $WD ]; then
    rm -r $WD; else
    mkdir $WD
fi

# Run singularity
singularity exec -B ${WORKDIR}:${WORKDIR} ${BRAKER_SIF} braker.pl \
--genome=$WORKDIR/scaffold_myo_myo.fasta \
--prot_seq=$WORKDIR/filtered_mammalia.fasta \
--softmasking \
--workingdir=${WORKDIR} \
--threads $SLURM_NTASKS \
--gff3
```
Simple, right? Let's break it down. It is crucial that you understand what each of the arguments do, and are aware of other available options. You can find a large list of those arguments by running `singularity exec -B ${PWD}:${PWD} ${BRAKER_SIF} braker.pl --help`.

The `INPUT FILE OPTIONS` section contains all arguments related to the data you will give BRAKER3 to do the genome annotation. The first option is, of course, the genome assembly, which is specified with the `--genome` argument. The `--bam` option can be used if you had BAM files of RNA-Seq alignments, while `--prot_seq` if you have some proteins. The choice of proteins will make a huge impact on annotation. Although BRAKER3 does a fairly good job when using protein evidence of distantly related species, using proteins from closely related ones is generally preferred. The `--hints` argument is utilized when there is about the intron structure of the species (or closely related ones), or if a gene model was trained with another software beforehand. The last arguments for data input are for raw RNA-Seq which are automatically downloaded from the [SRA](https://www.ncbi.nlm.nih.gov/sra). I encountered some problems with those flags when I tried to use them, so we will not cover it in class. However, I was able to annotate the scaffold use as example with RNA-Seq data from the liver, brain and kidney of *Myotis myotis*. You will examine those results later for comparison.

The other sections of the help page contain arguments related to how you want the algorithm to behave. I reccommend you have a close look to all of them, especially if you are performing a genome annotation in non-model organisms. The arguments we are using from all of those options are `--softmasking` which indicates the gene finders that we have marked repetitive regions in the DNA with lowercase letters (a,t,c,g); `--threads` to speed up the process using all cores available. I use `$SLURM_NTASKS` in there because it takes the number that we provide in `#SBATCH --ntasks=36` and uses all those processors by default. Finally, `--gff3` is a personal preference to get a `GFF3` file besides the `GTF` file that BRAKER3 gives as default. For me, it is more convenient to transform GFF3 files into various formats and extract relevant information. Each line in both files contains information about a feature (i.e., gene, CDS, exon, intron, etc), and each column contains information about the predicted features. You can find what those columns contain in [this link](https://useast.ensembl.org/info/website/upload/gff.html), and a lot more in-depth [here](https://github.com/The-Sequence-Ontology/Specifications/blob/master/gff3.md).

This example took around 7 hours to complete, so make sure you start working on it soon!
If you are wondering wether or not BRAKER3 completed its job succesfully, besides the `braker.gtf`, `braker.aa` and `braker.codingseq` files you will find in your `myo_myo` directory, you can also open the `err` file and will notice in the last lines the following:

```
[Wed Nov 22 05:57:01 2023] Finished spliced alignment
[Wed Nov 22 05:57:02 2023] Flagging top chains
[Wed Nov 22 05:57:03 2023] Processing the output
[Wed Nov 22 05:57:07 2023] Output processed
[Wed Nov 22 05:57:07 2023] ProtHint finished.
```
## For you to do

Once the genome annotation has completed, use the skills you have acquired in this course to answer the questions below. All these questions can be answered with the commands you have learned in this course. If you need a refresher, go to [Tutorial 1, Logging In to HPCC and an Intro to Bash and Linux Navigation](https://github.com/davidaray/Genomes-and-Genome-Evolution/wiki/01.-Logging-In-to-HPCC-and-an-Intro-to-Bash-and-Linux-Navigation).

1. How many genes were annotated in this scaffold of the *Myotis myotis* genome?

2. Based on your answer to the previous question, what is the gene density of this scaffold? That is, gene number/genome size. In this case, this scaffold is 223,369,599 bp long.

3. Are all of them protein coding genes? You can compare the braker.aa and braker.gff3 files to answer this question.

4. Because of time limits we couldn't work on functional annotation of the genes, but you can still get an idea of what functions the annotated genes may have. Use the `braker.aa` file and randomly pick a protein sequence (Please avoid using the first genes listed, explore the file a bit more). You can use the [BLASTp](https://blast.ncbi.nlm.nih.gov/Blast.cgi?PAGE=Proteins) tool to search for homologous proteins. Use the default parameters for the BLAST search, so just copy the protein sequence and let the web-tool do its work.
From my experience, though, it is way better to use the [InterProScan](https://www.ebi.ac.uk/interpro/search/sequence/) tool to search for protein domains. Use the same protein you used for the BLAST search and compare both results.
Now, if you were to report this gene in a paper, what would you say about it? How many exons and introns does it have? What's the length of that gene? What's its most likely function?
Please attach a screenshot of the NCBI and InterPro searches, and anything else you used to come up with your answer, including the name of the gene and the position of the genome it is located in.

5. Copy the files I obtained from the genome annotation using proteins and RNA-Seq data, and answer the same questions as in 1 and 2. You can get these files with: `cp /lustre/work/frcastel/software/braker/myo_myo_rna/braker_rna.* .`. Are the annotations derived from the combination of genome and proteins comparable to those resulting from the combination of genome, proteins, and RNA-Seq data? Which of both would you trust more? Why?


##  Sources and further reading

Understanding what is happening behind the scenes is fundamental to get the big picture of genome annotation. I highly recommend you read the [BRAKER3 paper](https://www.biorxiv.org/content/10.1101/2023.06.10.544449v3) to get a better grasp of the process. You should also read the [BRAKER3 GitHub repository](https://github.com/Gaius-Augustus/BRAKER#f19) to familiarize a lot more with its utilities. Even better, [here's a great lecture]((https://www.youtube.com/watch?v=UXTkJ4mUkyg&t=2967s&ab_channel=BiodiversityGenomicsAcademy)) by one the authors of BRAKER3, in which most of the details of their software are covered. Additionally, [this page](https://gatech-genemark.github.io/GeneMark-E-Docs/#/) has tons of information and details about how to prepare protein evidence, how to evaluate the annotation, and much more that we couldn't cover in class.

Eventhough the gene training is done automatically by BRAKER3, you can get a lot more control of the process if you run it using [Augustus](https://bioinf.uni-greifswald.de/augustus/) first, you can venture yourself in trying to get some evidence *a priori*, to incorporate into BRAKER3 later on. [This tutorial](https://vcru.wisc.edu/simonlab/bioinformatics/programs/augustus/docs/tutorial2015/training.html) is straightforward, although it won't show you how to install it, and that can be a long process most of the times. You can read the [AUGUSTUS paper](https://academic.oup.com/bioinformatics/article/26/8/1129/193348) and the [GitHub repository](https://github.com/Gaius-Augustus/Augustus).
