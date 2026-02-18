# Bacterial genome assembly - Advanced

⛔ This exercise assumes the following:
* A native Linux environment / terminal
* A functional micromamba installation
* Ability to install environments from
.yaml files
* Reads of a few bacterial genomes
* All files are stored within the home directory for the
`btg` user. Feel free to change locations according to own needs!

## Authors
These exercises where authored and tested by **Povilas Matusevicius** and **Kasper Thystrup Karstensen**

## Introduction

The goal of this practical is to help building a pipeline for handling bacterial data from raw reads to sequence type determination. The practical is build to hold you in the hand, while guiding you throughout the process.

⚠️ Therefore it is recommended to attempt the Intermediate exercise and only consult this exercise to guide your progress through the intermediate exercises.

## Prerequisites

For this exercise we will use the `BTG_QC` , `BTG_spades_4.2.0` , `BTG_alignment` environments. In addition, we will use the
following files and file paths.
* Path for read mate 1: `/home/btg/BTG/SequenceData/Ec016.illumina_R1.fastq.gz`
* Path for read mate 2: `/home/btg/BTG/SequenceData/Ec016.illumina_R2.fastq.gz`
* Path to Output folder: `/home/btg/BTG/Day8_pipelines/bacterial_asembly`

### Executing commands from different environments
A drawback of using conda/mamba environments, is that not all software can run in the same environment. When
running bash scripts, the shell doesn’t always know how to invoke the conda/mamba commands. With `micromamba`
though, there is a hack that can be utilized: Micromamba allow for cherry picking commands from different
environments using `micromamba run`. This way environments are not required to be loaded. So in order to run e.g.
MLST from within any (or no) environment, the following command can be used in the bash script.

```bash
micromamba run -n BTG_alignment mlst
```

## Bacterial pipeline
In the exercise, your task is to write a script called “bacterial_assembly.sh” that will provide a QC report, trim raw
reads, generate an assembly for sample, and run mlst on "Ec016" it.

### Setting up parameters
1. Navigate to the `Day8_pipelines` folder in `BTG` directory
2. Make a script file called `bacterial_assembly.sh` and open it.

💡 If you use nano to make the file, you don’t have to open it afterwards!

3. For this pipeline we will provide an argument parser to make execution easier for other users as well. Since
adding an argument parser to bash script is out of the scope for this Course, copy paste the following into the
empty `bacterial_assembly.sh` file:
```bash
#!/bin/bash
# Define usage function
usage() {
    echo "Usage: $(basename "$0") [-r|--read1 Read mate 1 file] [-R|--read2 Read mate 2 file] [-o|--output_dir Output directory]"
    exit 1
}

# Parse options
while [[ $# -gt 0 ]]; do
    # Making named positional arguments
    case "$1" in
        -r|--read1)
        read1="$2"
        shift
        ;;
        -R|--read2)
        read2="$2"
        shift
        ;;
        -o|--output_dir)
        output_dir="$2"
        shift
        ;;
        *)
        usage
        ;;
    esac
    shift
done
# Check if required arguments are provided
if [[ -z $read1 || -z $read2 || -z $output_dir ]]; then
    usage
fi

# Determine sample name from read mate 1 filename
read1_filename=$(basename $read1)
sample_name=${read1_filename%.illumina_R1.fastq.gz}
```

3. With the input and output settings determined, lets move one with the basic skeleton for handling the pipeline
output. Define the following output directories to the file, and manually write code for generating folders for
each of these folders:
* `fastqc_out=$output_dir/$sample_name/fastqc`
* `fastp_out=$output_dir/$sample_name/fastp`
* `spades_out=$output_dir/$sample_name/spades`
* `mlst_out=$output_dir/$sample_name/mlst`
* `results_dir=$output_dir/Results`

5. Add echo statement that prints out the value of `read1` and `sample_name`

6. Save the file and add execution permission for the script (`chmod u+x` )

7. Execute the script by running `./bacterial_assembly.sh` and ensure that the argument parser functions as expected,
and that the print statements work correctly.

### Read trimming
To ensure that we get a great overview of quality parameters we will impose semi-strict filtration criteria using `fastp` introduced during the Quality Assurance on Illumina Reads exercises.

1. Execute fastp by adding `micromamba run -n BTG_QC CMD` interchanging CMD with `fastqc` or `fastp`

💡 You can use `micromamba run -n BTG_QC fastp --help` to see help message for fastp. You should use this example
for all the tools you will be running to learn how to provide proper input.

2. Add fastp to the script, and add the following parameters

**Fastp**
* `-i` - read1 input
* `-o` - read1 output, recommend file name containing sample name and ending with _trimmed_R1.fastq.gz to show
* that it was trimmed
* `-I` - read2 input
* `-O` - read2 output, recommend file name containing sample name and ending with _trimmed_R1.fastq.gz to
* show that it was trimmed
* `--html $fastp_out/$sample_name.html`
* `--json $fastp_out/$sample_name.json`
* `--disable_adapter_trimming`
* `--length_required 100`
* `--qualified_quality_phred 30`
* `--trim_front1 9`
* `--trim_tail1 1`
* `--correction`
* `--overrepresentation_analysis`
* `--overrepresentation_sampling 10`

3. Execute the pipeline to ensure everything is still working

🎓 Pro advice: There are many steps, and it is easy to make a typing errors (some of the commands are very long!). So make your script one step at the time, and check that it works, before moving on to the next step. This can most easily be achieved by having two terminal open simultaneously, both with the loaded environment. One terminal handles the coding, while the other handles execution. Remember, you can easily disable commands in your script simply by adding a comment symbol ( # ) at the start of the line. Once you are ready to include the commands again, remove the comment symbol again.

The script so far ⚠️ Don’t read unless needing help!
```bash
#!/bin/bash
# Define usage function
usage() {
    echo "Usage: $(basename "$0") [-r|--read1 Read mate 1 file] [-R|--read2 Read mate 2 file] [-o|--output_dir Output directory]"
    exit 1
}

# Parse options
while [[ $# -gt 0 ]]; do
    # Making named positional arguments
    case "$1" in
        -r|--read1)
        read1="$2"
        shift
        ;;
        -R|--read2)
        read2="$2"
        shift
        ;;
        -o|--output_dir)
        output_dir="$2"
        shift
        ;;
        *)
        usage
        ;;
    esac
    shift
done
# Check if required arguments are provided
if [[ -z $read1 || -z $read2 || -z $output_dir ]]; then
    usage
fi
# Determine sample name from read mate 1 filename
read1_filename=$(basename $read1)
sample_name=${read1_filename%.illumina_R1.fastq.gz}
# Generating output folders
fastqc_out=$output_dir/$sample_name/fastqc
fastp_out=$output_dir/$sample_name/fastp
spades_out=$output_dir/$sample_name/spades
mlst_out=$output_dir/$sample_name/mlst
results_dir=$output_dir/Results
mkdir -p $fastqc_out
mkdir -p $fastp_out
mkdir -p $spades_out
mkdir -p $mlst_out
mkdir -p $results_dir
micromamba run -n BTG_QC fastp -i $read1 -o $fastp_out/"$sample_name"_trimmed_R1.fastq.gz -I $read2 -O $fastp_out/"$sample_name"_trimmed_R2.fastq.gz --html $fastp_out/$sample_name.html --json $fastp_out/$sample_name.json --disable_adapter_trimming --length_required 100 --qualified_quality_phred 30 --trim_front1 9 --trim_tail1 1 --correction --overrepresentation_analysis --overrepresentation_sampling 10
```

### Adding the assembler to the pipeline

After you trimmed low quality reads and determined that the general quality of reads will suffice, you need to
make an assembly. For this purpose you can use various assemblers, some of the popular ones are **spades**,
**skesa**, **unicycler** and many others.
1. For this task we will be using SPAdes, expand you script by introducing a `micromamba run` command, point to the
`BTG_spades_4.2.0` environment and execute the assembler using `spades.py`
2. Manually add the following parameters:
* `--isolate`
* `-1` - trimmed reads1
* `-2` - trimmed reads2
* `-o` - output folder

### Determine sequence type 
One of the more useful result after you have assembled the sequence is to determine the sample MLST.
1. Run MLST through the micromamba run command in the BTG_alignment environment with the mlst command
2. Manually add the following parameters:
* `--quiet`
* `--label $sample_name`
* `> $mlst_out/$sample_name.tsv` # Captures message in terminal into a tsv file

### Results
Lets collect all relevant information in a Results folder, so they are easily accessible from the Results directory.
Here we will copy important results files to the Results directory
1. Add the following commands to the end of the file
* `cp $fastp_out/"$sample_name"_trimmed_*.fastq.gz $results_dir/.`
* `cp $spades_out/contigs.fasta $results_dir/$sample_name.fasta`
* `cp $mlst_out/$sample_name.tsv $results_dir/.`
* `echo All done!`

**Congratulations on your very own Bacterial assembly pipeline!**

## Solution
```bash
#!/bin/bash
# Define usage function
usage() {
    echo "Usage: $(basename "$0") [-r|--read1 Read mate 1 file] [-R|--read2 Read mate 2 file] [-o|--output_dir Output directory]"
    exit 1
}

# Parse options
while [[ $# -gt 0 ]]; do

    # Making named positional arguments
    case "$1" in
        -r|--read1)
        read1="$2"
        shift
        ;;
        -R|--read2)
        read2="$2"
        shift
        ;;
        -o|--output_dir)
        output_dir="$2"
        shift
        ;;
        *)
        usage
        ;;
        esac
    shift
done
# Check if required arguments are provided
if [[ -z $read1 || -z $read2 || -z $output_dir ]]; then
    usage
fi
# Determine sample name from read mate 1 filename
read1_filename=$(basename $read1)
sample_name=${read1_filename%.illumina_R1.fastq.gz}
# Generating output folders
fastqc_out=$output_dir/$sample_name/fastqc
fastp_out=$output_dir/$sample_name/fastp
spades_out=$output_dir/$sample_name/spades
mlst_out=$output_dir/$sample_name/mlst
results_dir=$output_dir/Results
mkdir -p $fastqc_out
mkdir -p $fastp_out
mkdir -p $spades_out
mkdir -p $mlst_out
mkdir -p $results_dir
micromamba run -n BTG_QC fastp -i $read1 -o $fastp_out/"$sample_name"_trimmed_R1.fastq.gz -I $read2 -O $fastp_out/"$sample_name"_trimmed_R2.fastq.gz --html $fastp_out/$sample_name.html --json $fastp_out/$sample_name.json --disable_adapter_trimming --length_required 100 --qualified_quality_phred 30 --trim_front1 9 --trim_tail1 1 --correction --overrepresentation_analysis --overrepresentation_sampling 10
micromamba run -n BTG_spades_4.2.0 spades.py --isolate -1 $fastp_out/"$sample_name"_trimmed_R1.fastq.gz -2 $fastp_out/"$sample_name"_trimmed_R2.fastq.gz -o $spades_out
micromamba run -n BTG_alignment mlst "$spades_out"/contigs.fasta --quiet --label $sample_name > $mlst_out/$sample_name.tsv
# Generate a report on output and collect relevant files
cp $fastp_out/"$sample_name"_trimmed_*.fastq.gz $results_dir/
cp $spades_out/contigs.fasta $results_dir/$sample_name.fasta
cp $mlst_out/$sample_name.tsv $results_dir/
echo All done!
```

## Run the pipeline with sample directory as input
Having to point to individual read file is far from optimal, so in order to automate things further, we can introduce a for loop.
Here, we simplify the argument parser to only take in a directory with read files as input and run all tools on all the fastq pairs.

1. In the argument parer at the top of the script, replace all instances of `-r|--read1` with `-r|read_dir` and update the
`read1` variable to be `read_dir`
2. Update the message in the argument parser usage descriptor (e.g. `[-r|--read_dir REPLACE WITH SOMETHING HELPFUL!]`
3. Remove `-R|--read2` and the `read2` variable from the code
4. Insert the following code line after the argument parser:
```bash
read1_files=$(find $read_dir -maxdepth 1 -type f -name *_R1.*f*q* | sort)
read2_files=${read1_files%_R1.fastq.gz}_R2.fastq.gz
```
5. Integrate a for loop iterating on `read1_files` (Hint: call the for loop variable `read1` - We will ignore the `read2_files`
variable until the very end!)
6. Define the now missing `read2` variable with the following:

```bash
# Determine read mate 2
read2=${read1%_R1.fastq.gz}_R2.fastq.gz
```
7. End the for loop with done after copying the result files to the $results_dir
8. Add the fastqc command after the loop have ended. Manually add the following parameters
FastQC
* `-o $fastqc_out`
* `--memory 2048`
* `--threads 6`
* `--quiet`
Add $read1_files $read2_files to the end of the command
9. Run **MultiQC** through the `micromamba run` command in the `BTG_QC` environment with the `multiqc` command.
Manually add the following parameters:
* `-o $results_dir`
* `-qf`
* Point to the `$output_dir` as input
Done!

## Solution
```bash
#!/bin/bash
# Define usage function
usage() {
    exit 1
    echo "Usage: $(basename "$0") [-r|--read_dir Direcotry with read files] [-o|--output_dir Output directory]"
}
# Parse options
while [[ $# -gt 0 ]]; do
    # Making named positional arguments
    case "$1" in
        -r|--read_dir)
        read_dir="$2"
        shift
        ;;
        -o|--output_dir)
        output_dir="$2"
        shift
        ;;
        *)
        usage
        ;;
    esac
    shift
done
# Check if required arguments are provided
if [[ -z $read_dir || -z $output_dir ]]; then
usage
fi
read1_files=$(find $read_dir -maxdepth 1 -type f -name *_R1.*f*q* | sort)
read2_files=${read1_files%_R1.fastq.gz}_R2.fastq.gz
for read1 in $read1_files; do
    
    # Determine read mate 2
    read2=${read1%_R1.fastq.gz}_R2.fastq.gz
    
    # Determine sample name from read mate 1 filename
    read1_filename=$(basename $read1)
    sample_name=${read1_filename%.illumina_R1.fastq.gz}
    
    # Generating output folders
    fastqc_out=$output_dir/fastqc
    fastp_out=$output_dir/$sample_name/fastp
    spades_out=$output_dir/$sample_name/spades
    mlst_out=$output_dir/$sample_name/mlst
    results_dir=$output_dir/Results
    
    mkdir -p $fastqc_out
    mkdir -p $fastp_out
    mkdir -p $spades_out
    mkdir -p $mlst_out
    mkdir -p $results_dir
    
    micromamba run -n BTG_QC fastp -i $read1 -o $fastp_out/"$sample_name"_trimmed_R1.fastq.gz -I $read2 -O $fastp_out/"$sample_name"_trimmed_R2.fastq.gz --html $fastp_out/$sample_name.html --json $fastp_out/$sample_name.json --disable_adapter_trimming --length_required 100 --qualified_quality_phred 30 --trim_front1 9 --trim_tail1 1 --correction --overrepresentation_analysis --overrepresentation_sampling 10
    micromamba run -n BTG_spades_4.2.0 spades.py --isolate -1 $fastp_out/"$sample_name"_trimmed_R1.fastq -2 $fastp_out/"$sample_name"_trimmed_R2.fastq -o $spades_out
    micromamba run -n BTG_alignment mlst "$spades_out"/contigs.fasta --quiet --label $sample_name > $mlst_out/$sample_name.tsv
    
    # Collect relevant files
    cp $fastp_out/"$sample_name"_trimmed_*.fastq.gz $results_dir/
    cp $spades_out/contigs.fasta $results_dir/$sample_name.fasta
    cp $mlst_out/$sample_name.tsv $results_dir/
done

# Generate a summary report
micromamba run -n BTG_QC fastqc -o $fastqc_out --memory 2048 --threads 6 --quiet $read1_files $read2_files
micromamba run -n BTG_QC multiqc -o $results_dir -qf $output_dir

echo All done!
```

Finally. Take the pipeline for a spin, execute using the following user input -r ~/BTG/SequenceData -o
bacterial_assembly