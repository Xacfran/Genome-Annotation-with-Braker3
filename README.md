# Genome Annotation
[![hackmd-github-sync-badge](https://hackmd.io/jqXB_tEgQtWxRVZc1ChNeQ/badge)](https://hackmd.io/jqXB_tEgQtWxRVZc1ChNeQ)

###### tags: `genome annotation` `singularity` `BRAKER3`

###### Last updated: Nov 22, 2023

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

The first file is the largest scaffold of the genome assembly of the bat *Myotis myotis* and the second one is a filtered version of the [Vertebrata dataset](https://bioinf.uni-greifswald.de/bioinf/partitioned_odb11/) from [OrthoDB](https://www.orthodb.org/).

## <i class="fa fa-gears fa-lg"></i> Installation

To get the BRAKER3 SIF file you need to run:

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

This will download 3 test scripts that will run BRAKER3 in different modes. Fore this tutorial we will only use the first one, but you can try the others on your own. You can run the script with `bash test1.sh`.

After some minutes you will see a directory called `test1` with the results of the annotation. You can take a look at the files with `ls test1`. If you see the files `braker.gtf`, `braker.aa` and `braker.codingseq`, and if they are not empty, then everything worked fine. Now moving on with our own data.

### Annotating a bat genome assembly

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
wd=myo_myo

if [ -d $wd ]; then
    rm -r $wd; else
    mkdir $wd
fi

# Run singularity
singularity exec -B ${PWD}:${PWD} ${BRAKER_SIF} braker.pl --genome=/path/to/your/scaffold_myo_myo.fasta --prot_seq=/path/to/your/filtered_mammalia.fasta --softmasking --workingdir=${wd} --threads $SLURM_NTASKS --gff3
```
If you want to understand what each of these commands does, and others available, you can type `singularity exec braker3.sif braker.pl --help`. This will show you the help menu of BRAKER3.

This example took around 7 hours to complete. Besides the `braker.gtf`, `braker.aa` and `braker.codingseq` files you will find in your `myo_myo` directory, you can also see in the last lines of your err file something like:

```
[Wed Nov 22 05:57:01 2023] Finished spliced alignment
[Wed Nov 22 05:57:02 2023] Flagging top chains
[Wed Nov 22 05:57:03 2023] Processing the output
[Wed Nov 22 05:57:07 2023] Output processed
[Wed Nov 22 05:57:07 2023] ProtHint finished.
```

##  Sources and further reading

