LIONS pipeline architecture
#====================================================================
 LIONS is a bioinformatic analysis pipeline which brings together a
 few pieces of software and some home-brewed scripts to annotate a
 paired-end RNAseq library against a reference TE annotation set
 (such as Repeat Masker).

 East Lion proccesses a bam file input, re-aligns it to a genome,
 builds an ab initio assembly using Tophat2. This assembly is then
 proccessed and local read searches are done at the 5' ends to find
 additional transcript start sites and quality control the 5' ends of
 the assembly. The output is a file-type <library>.lions which
 annotates the intersection between the assembly, a reference gene set
 and repeat set.

 West Lion compiles different .lions files, groups them into biological
 catagories (i.e. Cancer vs. Normal or Treatment vs. Control) and
 compares and analyzes the data to create graphs and meaningful
 interpretation of the data.

# LIONS File Architecture
# ===================================================================
# ./
	base directory:  LIONS is a self-contained pipeline and 
	references needed by LIONS from the system are linked
	within this directory.

	./lions.sh <parameter.ctrl>
		The master script from which the entire pipeline is ran
		The script reads and procceses all files in input.list
		and all parameters can be controlled from parameter.ctrl

#./controls
	Control Files: This folder contains project and system-specific
	parameters for running LIONS. There are three main files which
	need to be set-up, LIONS run parameters, system parameters and
	input RNA-seq libraries.
	

	./controls/parameter.ctrl
		A bash script which defines global project-specific 
		variables such as Project Name, library input list etc...
		The .sysctrl and .list file are defined here as well

	./controls/system.sysctrl
		A bash script which defines global variables for all LIONS
		scripts. Also defines system-specific variables such as
		System Name, number of CPU cores etc...

	./controls/input.list
		A three column tab-delimited file defining:
		<Library_Name> <Library_Path_on_system> <Biological_Grouping>
		Group Naming Convention: 1 = control 2 = experimental 3 = other
		
	
# ./bin
	Binary: A folder for symbolic links to binaries needed by
	the pipeline and script to initilize the folder. Make sure to set
	the correct commands for the software list in parameters.ctrl for
	your system


# ./projects
	For each <Project_Name> a single folder will be initilized in which
	the data will be organized.

	./projects/<Project_Name>
	The main directory for this project. Each individual library in the
	input will have a folder generated here called <Library>.

	./projects/<Project_Name>/logs
	Folder contains run-specific information such as input file at time
	of run and a copy of the input parameter file at time of run.

	./projects/<Project_Name>/Analysis(_RUNID)
	Not implemented yet. This contains all data analysis for a run
	of LIONS. All graphs and project-wide .lions files are stored here
	
	./projects/<Project_Name>/<Library>
	Library-specific data and primary analysis files.

	./projects/<Project_Name>/<Library>/<Library>.lcsv
	Raw output file from LIONS containing all possible TE-Exon interaction
	data. This will include initiations, exonizations and terminations
	along with many calculated values about these loci from which
	LIONS will sort initiation events from the others. <Library>.pc.lcsv
	is the same file with additional information about overlapping protein
	coding genes.

	./projects/<Project_Name>/<Library>/<Library>.lions
	Initiation only TE-exon data from post-sorting. This file is the 
	complete list of transcripts initiated by TEs in this library. This
	data is passed on to the West Lion protocol to compare TE usage
	between libraries.

	./projects/<Project_Name>/<Library>/alignment
	tophat2-generated alignment and the re-aligned .bam file which
	will be used for analysis. Also contains flagstats and log files.
	Note: Once the alignment is generated it will not be re-generated
	even if you change the alignment parameters in parameter.ctrl. To
	re-make alignments simply start the project with a new name or delete
	these files.

	./projects/<Project_Name>/<Library>/assembly
	cufflinks-generated assembly in 'transcripts.gtf'

	./projects/<Project_Name>/<Library>/expression
	The output from a series of custom scripts 'RNAseqPipeline' which
	will generate wig files and perform RPKM calculations on a series 

	./projects/<Project_Name>/<Library>/resources
	Charlie-foxtrot of library-specific files used to calculate a score
	of parameters in the pipeline.

# ./scripts
	Scripts: all scripts to run lions are held here except for the
	controlling lions.sh script which is in the base folder.
	Check initializeScripts.sh for complete list of scripts
	The main scripts are 

	./scripts/eastLion.sh
		Alignment, Assembly, Chimeric Detection pipeline

	./scripts/westLion.sh
		LIONS analysis pipeline

# ./resources
	<INDEX>: the name of the index set. To be compatible with different
	genome versions, species and gene sets there can be different sets
	of data. The <INDEX> global variable is set in parameter.ctrl file
	
	./resources/<INDEX>/annotation
	<GENESET> file: A ucsc formatted file containing a gene set to be
	used in the analysis (i.e. look for overlapping genes to transcripts)
	Download from: https://genome.ucsc.edu/cgi-bin/hgTables
	The standard annotation set used was RefSeq 'RefGene' table.

	./resources/<INDEX>/genome
	<INDEX>.fa: The only requisite file here for running LIONS is a fasta
	formatted genome. This could be a symbolic link. LIONS will generate
	the other files neccesary from INDEX.fa. If you have the .bt2 index
	files already generated you can symbolically link them in this folder
	to skip re-generating them.

	./resources/<INDEX>/repeat
	<REPEATMASKER>.ucsc: a ucsc formatted file containing the repeatMasker
	annotation for the genome.
	(Download from UCSC genome browswer or format from RepeatMasker)
	Columns are;
	bin, swScore, milliDiv, milliDel, milliIns, genoName, genoStart,
	genoEnd, genoEnd, genoLeft, strand, repName, repClass, repFamily,
	repStart, repEnd, repLeft, id

# ./software
	Packaged with LIONS is a few bits of software which will set-up your
	system to run the pipeline. Namely setuptools and pysam are the most
	challenging things to install. I found that it's easiest to set-up
	the pipeline using pip and download the package pysam from there.
	Pysam is used to read teh bam files in the python scripts.






