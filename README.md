# Genome Annotation
[![hackmd-github-sync-badge](https://hackmd.io/jqXB_tEgQtWxRVZc1ChNeQ/badge)](https://hackmd.io/jqXB_tEgQtWxRVZc1ChNeQ)

###### tags: `genome annotation` `singularity` `BRAKER3`

###### Last updated: Nov 22, 2023 
###### This still work in progress

------------------------------------------------------------------------

This tutorial was written for students of the Genomes and Genome Evolution course at Texas Tech and tested at the [High Performance Computing Cluster](https://https://www.depts.ttu.edu/hpcc/) (HPCC) of this University.

## <i class="fa fa-download fa-lg"></i> Pre-requisites

You will only need to have Singularity installed. Singularity is a tool that allows you to create and run applications in containers. What are containers you may ask? Of course the name gives away what they do, they contain things. In this case, all the software and dependencies you need to run a specific application. This is AWESOME because you only need to have a **S**ingularity **I**mage **F**ormat (SIF) and anything else. This is great because otherwise it may take you months to install everything you need, I know this from experience. Take a look at [this](https://github.com/Gaius-Augustus/BRAKER#braker) and you'll see everything you would need to manually install to use BRAKER3 to annotate a genome.

 Luckily for us, Singularity is already installed at the HPCC. If you are using your own computer, you can follow the instructions [here](https://docs.sylabs.io/guides/3.0/user-guide/installation.html) to install it.

Lastly, the files you will use in this tutorial are available at my [GitHub repository](https://github.com/Xacfran/Genome-Annotation-with-Braker3) if you are not a Tech student. If you are working on the HPCC, you can copy them to your own directory with:

```bash
cp /lustre/work/frcastel/software/braker/scaffold_myo_myo.fasta .
cp /lustre/work/frcastel/software/braker/filtered_mammalia.fasta .
```

The first file is the largest scaffold of the genome assembly of the bat *Myotis myotis* you worked with some weeks ago, and the second one is a filtered version of the [Vertebrata dataset](https://bioinf.uni-greifswald.de/bioinf/partitioned_odb11/) from [OrthoDB](https://www.orthodb.org/). We'll use this dataset to train the gene models, but first lets get the file that will allow us to use BRAKER3.

## <i class="fa fa-gears fa-lg"></i> Installation

To get the BRAKER3 SIF file you simply need to run:

```bash
singularity build braker3.sif docker://teambraker/braker3:latest
```

## <i class="fa fa-code fa-lg"></i> Usage

BRAKER3 runs in different modes, it can make use of the genome alone, genome + proteins, genome + RNA-seq, or all of them. You can find the details of how each mode works [here](https://github.com/Gaius-Augustus/BRAKER#overview-of-modes-for-running-braker). In this tutorial we will use proteins, but first lets use some test scripts and data to make sure it works.

### Test data

```bash
singularity exec -B $PWD:$PWD braker3.sif cp /opt/BRAKER/example/singularity-tests/test1.sh .
singularity exec -B $PWD:$PWD braker3.sif cp /opt/BRAKER/example/singularity-tests/test2.sh .
singularity exec -B $PWD:$PWD braker3.sif cp /opt/BRAKER/example/singularity-tests/test3.sh .
```

This will download 3 test scripts that will run BRAKER3 in different modes. For this tutorial we will only use the first one, but you can try the others on your own. You can run the script with `bash test1.sh`.

After some minutes you will see a directory called `test1` with the results of the annotation. You can take a look at the files with `ls test1`. If you see the files `braker.gtf`, `braker.aa` and `braker.codingseq`, and if they are not empty, then everything worked fine. Now moving on with our own data.

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
Simple, right? Let's break it down. It is extremely important that you understand what each of the arguments in `braker.pl` do, and what others are available to you. You can find a large list of those arguments by running `singularity exec -B ${PWD}:${PWD} ${BRAKER_SIF} braker.pl --help`.

The `INPUT FILE OPTIONS` section contains all arguments related to the data you will give BRAKER3 to do the genome annotation. The first option is of course the genome assembly, which you provide with the `--genome` argument, followed by `--bam`, in case you had bam files of RNA-Seq alignments, and `--prot_seq`. The proteins you choose to use here will make a huge impact on the genome annotation, and eventhough BRAKER3 does a fairly good job when using protein evidence of distantly related species, it is always better to use proteins from closely related ones. The `--hints` argument will be used only if you have any sort information regarding the intron structure of the species (or closely related), and/or if you trained a gene model with any other software before. Last arguments available for data input are for raw RNA-Seq. I encountered some problems with it when I tried to use it, so we will not cover it in class but I made a genome annotation using the RNA-Seq data of *Myotis myotis* and you will be given the data to compare later.

The other sections contain arguments related to how you want the algorithm to behave and I reccommend you have a close look to all of them if you are performing a genome annotation of your own. Especially in non-model organisms. The arguments we are using from all of those options are `--softmasking` which lets the gene finders know that we have marked repetitive regions with low-case letters (a,t,c,g), `--threads` to speed up the process using all cores available. I use `$SLURM_NTASKS` because it takes the number that we provide in `#SBATCH --ntasks=36` and uses all those processors by default. Finally, `--gff3` is a personal preference to get a `gff3` besides the `gtf` file that BRAKER3 gives as default. For me, it is much easier to transform `gff3` files into any other format and to extract information that we are interested in.

This example took around 7 hours to complete, so make sure you start working on it soon! Besides the `braker.gtf`, `braker.aa` and `braker.codingseq` files you will find in your `myo_myo` directory, you can also see in the last lines of your err file something like:

```
[Wed Nov 22 05:57:01 2023] Finished spliced alignment
[Wed Nov 22 05:57:02 2023] Flagging top chains
[Wed Nov 22 05:57:03 2023] Processing the output
[Wed Nov 22 05:57:07 2023] Output processed
[Wed Nov 22 05:57:07 2023] ProtHint finished.
```
## For you to do

Once the genome annotation has completed, use the skills you have acquired in this course to answer the questions below. All questions can be answered with the commands you learned how to use in [Tutorial 1, Logging In to HPCC and an Intro to Bash and Linux Navigation](https://github.com/davidaray/Genomes-and-Genome-Evolution/wiki/01.-Logging-In-to-HPCC-and-an-Intro-to-Bash-and-Linux-Navigation).

1. How many genes were annotated in this scaffold of the Myotis myotis genome?

2. Based on your answer to the previous question, what is the gene density of this scaffold? That is gene number/genome size. In this case, this scaffold is 223,369,599 bp long.

3. Are all of them protein coding genes? You can compare the braker.aa and braker.gff3 files to answer this question.

4. Because of time limits we couldn't work on functional annotation of the genes, but you can still get an idea of what functions the annotated genes may have. Use the `braker.aa` file and randomly pick a protein sequence (Please avoid using the first gene "g1.t1"). you can use the [BLASTp](https://blast.ncbi.nlm.nih.gov/Blast.cgi?PAGE=Proteins) tool to search for homologous proteins in the [NCBI non-redundant protein database](https://www.ncbi.nlm.nih.gov/refseq/about/nonredundantproteins/). You can use the default parameters for the BLAST search, so just copy the protein sequence and let the web-tool do its work.
From my experience, though, it is way better to use the [InterProScan](https://www.ebi.ac.uk/interpro/search/sequence/) tool to search for protein domains. So, again use the same protein you blasted and search for protein domains in the InterPro web page.
Now, if you were to report this gene in a paper, what would you say about it? How many exons and introns does it have? What's the length of that gene? What's its most likely function?
Please attach a screenshot of the NCBI, Interpro and the braker.gff3 files to this answer.

5. Copy the files I obtained as resultf of the genome annotation of *Myotis myotis* using RNA-Seq data to your own directory with: `cp /lustre/work/frcastel/software/braker/myo_myo_rna/braker_rna.* .` and answer the same questions as in 1 and 2. You can also compare the results of the two annotations. Are they similar? Why do you think that is?

##  Sources and further reading

