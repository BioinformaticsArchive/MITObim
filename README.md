MITObim - mitochondrial baiting and iterative mapping
=====================================================



VERSION
-------
1.3

Copyright 2012 - Christoph Hahn

CONTACT
-------

Christoph Hahn <christoph.hahn@nhm.uio.no>

INTRODUCTION
------------

This document contains instructions on how to use the MITObim pipeline described in the manuscript "Reconstructing mitochondrial genomes directly from genomic next generation sequencing reads - a baiting and iterative mapping approach" by Hahn et al., submitted to NAR methods online. The pipeline is at the moment intended to be used with illumina data, but can be readily modified for the use with other platforms data. The latest version of the wrapper script (including the proofreading function) has been uploaded 31.12.2012. The tutorial will be updated accordingly early 2013. 


PREREQUISITES
-------------

- GNU utilities
- perl
- A running version of MIRA v3.4.1.1 (http://sourceforge.net/projects/mira-assembler/files/MIRA/stable/) is required. An excellent guide to MIRA v3.4 is available at http://mira-assembler.sourceforge.net/docs/DefinitiveGuideToMIRA.html.


General introduction to MITObim
-------------------------------

The MITObim procedure (mitochondrial baiting and iterative mapping) represents a highly efficient approach to assembling novel mitochondrial genomes of non-model organisms directly from total genomic DNA derived NGS reads. Labor intensive long-range PCR steps prior to sequencing are no longer required. MITObim is capable of reconstructing mitochondrial genomes without the need of a reference genome of the targeted species by relying solely on (a) mitochondrial genome information of more distantly related taxa or (b) short mitochondrial barcoding sequences (seeds), such as the commonly used cytochochrome-oxidase subunit 1 (COI), as a starting reference. 

The script is performing three steps and iteratively repeating them: (i) Deriving reference sequence from previous mapping assembly, (ii) in silico baiting using the newly derived reference (iii) previously fished reads are mapped to the newly derived reference leading to an extension of the reference sequence.
For more details please refer to the manuscript. Detailed examples are demonstrated in the TUTORIALS section below.


TUTORIALS
---------

The following tutorials are designed for users with little Unix and no previous MIRA experience. Tutorials I & II will demonstrate how to recover the complete mitochondrial genome of _Thymallus thymallus_ using the mitochondrial genome of _Salvelinus alpinus_ as a starting reference. Tutorial III achieves the same goal using solely a ~700 bp barcoding sequence as initial seed reference. Tutorial IV (to be finished early 2013) uses a proofreading procedure to specifically reconstruct two mitochondrial genomes from a mixed sample containing genomic reads from two species. 

Preparations:

- download the MITObim wrapper script MITObim.pl and make it executable (chmod a+x MITObim.pl)
- download testdata1.tgz and testdata2.tgz and extract the contents (tar xvfz testdata?.tgz)

Test the wrapper script by doing:

	-bash-4.1$ ~/PATH/TO/MITObim.pl

which should display the usage:

	usage: ./MITObim.pl <parameters>

	parameters:

                -start <int>            iteration to start with, default=1
                -end <int>              iteration to end with, default=1
                -strain <string>        strainname as used in initial MIRA assembly
                -ref <string>           referencename as used in initial MIRA assembly
                -readpool <PATH>        path to readpool in fastq format
                -maf <PATH>             path to maf file from previous MIRA assembly


	optional:

                --denovo                runs MIRA in denovo mode, default: mapping
                --pair                  finds pairs after baiting, default: no
                --quick <PATH>          starts process with initial baiting using provided fasta reference
                --noshow                do not show output of MIRA modules
                --help                  shows this information
                --proofread             applies proofreading
                --readlength <int>      read length of illumina library, default=150
                --insert <int>          insert size of illumina library, default=300

	examples:

                ./MITObim.pl -start 1 -end 5 -strain StrainX -ref Gthymalli-mt -readpool /PATH/TO/readpool.fastq -maf /PATH/TO/assembly.maf
                ./MITObim.pl --quick /PATH/TO/reference.fasta -strain StrainY -ref Gthymalli-mt -readpool /PATH/TO/readpool.fastq

The archive testdata1 contains three files:

1. Tthymallus-150bp-300sd50-interleaved.fastq - 6000 simulated illumina reads (read length 150 bp, insert size 300 +- 50 bp) for the mitochondrial genome of _T. thymallus_ as discussed in Hahn et al.

2. Salpinus-mt-genome-NC_000861.fasta - mitochondrial genome of _S. alpinus_ in fasta format downloaded from Genbank (accession NC000861).

3. Tthymallus-COI-partial-HQ961018.fasta - partial COI sequence of _T. thymallus_ (Genbank acc. HQ961018).

The archive testdata2 (to be uploaded soon) contains three files:

1. pooled-150bp-300sd50-interleaved.fastq - mixed dataset containing 6000 simulated reads for the mt genomes of _T. thymallus_ (Genbank acc. FJ853655) and _S. alpinus_ (Genbank acc. NC_000861), respectively.

2. Tthymallus-COI-FJ853655.fasta - 1200 bp COI sequence of _T. thymallus_ extracted from the mt genome used for read simulation (Genbank acc. FJ853655).

3. Salpinus-COI-NC_000861.fasta - 1200 bp COI seqeuence of _S. alpinus_ extracted from the mt genome used for read simulation (Genbank acc. NC_000861).

TUTORIAL I: reconstruction of a mitochondrial genome using a two step procedure
-------------------------------------------------------------------------------

One strategy for successful MITObim performance is to prepare an initial reference from the conserved regions of a distantly related mitochondrial reference genome using a MIRA mapping assembly in a first step. In a second step the MITObim wrapper script is applied to this newly constructed reference in order to reconstruct the entire mitochondrial genome.

a. Initial mapping assembly using MIRA v3.4.1.1:

Create a directory to work in and change into this directory:

	-bash-4.1$ mkdir tutorial1
	-bash-4.1$ cd tutorial1

The project name of the initial mapping assembly might be e.g. "initial-mapping-testpool-to-Salpinus-mt". Copy the readpool and the reference from the testdata1 directory to your work directory and rename them so that MIRA will recognize the files.
  
	-bash-4.1$ cp /PATH/TO/testdata1/Tthymallus-150bp-300sd50-interleaved.fastq initial-mapping-testpool-to-Salpinus-mt_in.solexa.fastq 
	-bash-4.1$ cp /PATH/TO/testdata1/Salpinus-mt-genome-NC_000861.fasta initial-mapping-testpool-to-Salpinus-mt_backbone_in.fasta
  
Run a mapping assembly (details on the options can be found at http://mira-assembler.sourceforge.net/docs/DefinitiveGuideToMIRA.html):

	-bash-4.1$ mira --project=initial-mapping-testpool-to-Salpinus-mt --job=mapping,genome,accurate,solexa -MI:somrnl=0 -AS:nop=1 -SB:bft=fasta:bbq=30:bsn=Omykiss-mt-genome SOLEXA_SETTINGS -CO:msr=no -SB:dsn=testpool 

Look what happened:

	-bash-4.1$ ls -hlrt
	total 2.8M
	-rw-r--r-- 1 chrishah users  17K Oct 21 22:50 initial-mapping-testpool-to-Salpinus-mt_backbone_in.fasta
	-rw-r--r-- 1 chrishah users 2.7M Oct 21 22:51 initial-mapping-testpool-to-Salpinus-mt_in.solexa.fastq
	drwxr-xr-x 6 chrishah users    4 Oct 21 22:51 initial-mapping-testpool-to-Salpinus-mt_assembly

The successfull MIRA instance created a directory "initial-mapping-testpool-to-Salpinus-mt_assembly", containing four directories:

	-bash-4.1$ ls -hlrt initial-mapping-testpool-to-Salpinus-mt_assembly/
	total 2.0K
	drwxr-xr-x 2 chrishah users  2 Oct 21 22:51 initial-mapping-testpool-to-Salpinus-mt_d_chkpt
	drwxr-xr-x 2 chrishah users 15 Oct 21 22:51 initial-mapping-testpool-to-Salpinus-mt_d_tmp
	drwxr-xr-x 2 chrishah users  9 Oct 21 22:51 initial-mapping-testpool-to-Salpinus-mt_d_results
	drwxr-xr-x 2 chrishah users  8 Oct 21 22:51 initial-mapping-testpool-to-Salpinus-mt_d_info
 
The newly constructed reference is contained in the file "initial-mapping-testpool-to-Salpinus-mt_out.maf" in the "initial-mapping-testpool-to-Salpinus-mt_d_results" directory.

b. Baiting and iterative mapping using the MITObim.pl script: 

Run the wrapper script as follows (standard output will be written into the file log):

	-bash-4.1$ /PATH/TO/MITObim.pl -start 1 -end 10 -strain testpool -ref Salpinus_mt_genome -readpool ~/PATH/TO/tutorial1/initial-mapping-testpool-to-Salpinus-mt_in.solexa.fastq -maf ~/PATH/TO/tutorial1/initial-mapping-testpool-to-Salpinus-mt_assembly/initial-mapping-testpool-to-Salpinus-mt_d_results/initial-mapping-testpool-to-Salpinus-mt_out.maf &> log

NOTE: 
- The strain name needs to be the same as used in the initial MIRA assembly.
- The full path to the readpool and the maf file is required.

After the process has finished looking into the log file will yield something like:

	-bash-4.1$ tail log

	End of assembly process, thank you for using MIRA.

	readpool contains 6000 reads
	contig length: 16667

	MITObim has reached a stationary read number after 8 iterations!!

Allthough you have asked for 10 iterations MITObim stopped the process after 8 iterations because it has detected a stationary readnumber. It will have created 8 iteration directories. Each of which is organized like the working directory above (initial mapping assembly), containing a MIRA assembly directory plus the referernce and readpool used in the respective iteration:

	-bash-4.1$ ls -hlrt iteration1/
	total 2.3M
	-rw-r--r-- 1 chrishah users  17K Oct 21 23:45 testpool-Salpinus_mt_genome_backbone_in.fasta
	-rw-r--r-- 1 chrishah users 383K Oct 21 23:45 hashstat.bin
	-rw-r--r-- 1 chrishah users 1.9M Oct 21 23:45 testpool-Salpinus_mt_genome_in.solexa.fastq
	drwxr-xr-x 6 chrishah users    4 Oct 21 23:45 testpool-Salpinus_mt_genome_assembly

A fasta file containing the complete mitochondondrial genome of _T. thymallus_ can be found in the final iteration directory. See its contents by doing e.g.:

	-bash-4.1$ less iteration8/testpool-Salpinus_mt_genome_assembly/testpool-Salpinus_mt_genome_d_results/testpool-Salpinus_mt_genome_out.unpadded.fasta

**Congratulations!!!**

NOTE: For real data this strategy might require substantial computational resources depending on the size of the illumina readpool (Memory requirements can be estimated using the miramem program shipping with MIRA). This increased memory consumption can be bypassed by limitating the readpool e.g. via an initial in-silico baiting step using mirabait (in-silico baiting program shipping with MIRA). While this strategy will impair the initial sensitivity of the approach and might therefore require more iterations to finish, it works reasonably well when a not to distantly related reference is available. This strategy can be performed by using the --quick flag, together with providing a reference sequence in fasta format. A detailed example can be found in TUTORIAL II below.

TUTORIAL II - direct reconstruction without prior mapping assembly using the --quick option
-------------------------------------------------------------------------------------------
Run the MITObim.pl script with the --quick option, providing a reference in fasta format:

	-bash-4.1$ mkdir tutorial2
	-bash-4.1$ cd tutorial2
	-bash-4.1$ /PATH/TO/MITObim.pl -start 1 -end 30 -strain testpool -ref Salpinus_mt_genome -readpool ~/PATH/TO/testdata1/Tthymallus-150bp-300sd50-interleaved.fastq --quick ~/PATH/TO/testdata1/Salpinus-mt-genome-NC_000861.fasta &> log

You will find that with this approach MITObim will reconstruct the mitochondrial genome and reach a stationary number of mitochondrial reads only after 15 iterations. The result should nevertheless be equal to that obtained in TUTORIAL I.

TUTORIAL III - reconstructing mt genomes from mt barcode seeds
--------------------------------------------------------------

This tutorial reconstructs the mt genome of _T. thymallus_ solely using a partial mitochondrial COI sequence as starting seed.

	-bash-1.4$ mkdir tutorial3
	-bash-1.4$ cd tutorial3
	-bash-1.4$ ~/PATH/TO/MITObim.pl -strain testpool -ref Tthymallus-COI -readpool ~/PATH/TO/testdata1/Tthymallus-150bp-300sd50-interleaved.fastq --quick ~/PATH/TO/testdata1/Tthymallus-COI-partial-HQ961018.fasta -end 100 --noshow &> log

MITObim reconstructs the mitchondrial genome in 82 iterations.

For "well behaved" datasets the standard mapping assembly can be substituted by a _de novo_ assembly (--denovo flag). Utilizing read pair information (--paired flag) can further speed up the reconstruction if run in _de novo_ mode. This however will not decrease the nuber of necessary iterations in standard mapping mode.
Test this strategy, like so:

	-bash-4.1$ mkdir tutorial3-denovo
	-bash-4.1$ cd tutorial3-denovo
	-bash-4.1$ ~/PATH/TO/MITObim.pl -strain testpool -ref Tthymallus-COI -readpool ~/PATH/TO/testdata1/Tthymallus-150bp-300sd50-interleaved.fastq --quick ~/PATH/TO/testdata1/Tthymallus-COI-partial-HQ961018.fasta -end 50 --noshow --denovo --paired &> log

This strategy reconstructs the correct mitochondrial genome in only 31 iteration.



 

