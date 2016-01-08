RibORF: Identify translated ORFs using ribosome profiling

RibORF is a computational algorithm to identify genome-wide translated ORFs using ribosome profiling data, distinguishing off-frame ORFs and RNAs not associated with ribosomes.

Requirements: 
A standard R installation and the package ¡°e1071¡± for the Support Vector Machine classifier. Installations need to be in the PATH.

Steps to analyze ribosome profiling data and run RibORF:

1. Trim adapter of ribosome profiling reads.

perl removeAdaptor.pl fastqFile adapterSequence outputFile [readLengthCutoff]
    fastqFile: raw fastq read sequences;
    adapterSequence: 5' end sequence of adapter, 10nt is recommended;
    outputFile: output file;
    readLengthCutoff [optional]: read length cutoff after trimming adapters, default is 15 nt.

Example: perl removeAdaptor.pl ./example.data/sample.fastq CTGTAGGCAC ./example.data/adapter.sample.fastq 15

2. Map trimmed reads to ribosomal RNAs. Then non-ribosomal reads were aligned to transcriptomes and genomes.

Example:
bowtie -v 1 -k 2 ribosome_RNA.index --un=norrna.adapter.sample.txt -q adapter.sample.fastq ribosome.align.adapter.sample.txt
tophat -p 2 --no-convert-bam --GTF transcripts.gtf -o outDir genome.index norrna.adapter.sample.txt

3. Group reads based on fragment length, and check their 5' ends around start and stop codons of canonical protein-coding ORFs.

perl readDist.pl readFile geneFile outputDir readLength [leftNum] [rightNum]
    readFile: read mapping file, SAM format;
    geneFile: canonical protein-coding ORF annotation, genepred format;
    outputDir: output directory;
    readLength: specified RPF length;
    leftNum [optional]: N nucleotides upstream start codon and downstream stop codon, default: 30;
    rightNum [optional]: N nucleotides downstream start codon and upstream stop codon, default: 50.

Example: perl readDist.pl ./example.data/sample.mapping.sam ./example.data/hg19.coding.gene.txt ./example.data 30 30 50


4. Correct read locations based on offset distances between 5¡¯ ends and ribosomal A-sites.
Based on the distribution of read fragments around start and stop codon of canonical protein-coding ORFs, manually check the offset distance between 5¡¯ end and ribosomal A-site. Put correction parameters in a file, i.g. "offset.corretion.parameters.txt".
Note: Different ribosomal profiling experiments may have different offset correction parameters.

perl offsetCorrect.pl readFile offsetParameterFile readCorrectedFile;
    readFile: read mapping file before offset correction, SAM format;
    offsetParameterFile: parameters for offset correction, 1st column: read length, 2nd column: offset distance;
    readCorrectedFile: output file after offset correction. 

Example: perl offsetCorrect.pl ./example.data/sample.mapping.sam ./example.data/offset.corretion.parameters.txt ./example.data/corrected.sample.mapping.sam

5. Check the corrected read locations around start and stop codons of canonical protein-coding ORFs. 
This step is to check whether read distribution after offset correction shows clear 3-nt periodicity. 

perl readDist.pl readFile geneFile outputDir readLength [leftNum] [rightNum]

Example: perl readDist.pl ./example.data/corrected.sample.mapping.sam ./example.data/hg19.coding.gene.txt ./example.data 1 30 50

6. Run RibORF to identify translated ORFs.

perl ribORF.pl readCorrectedFile candidateORFFile outputDir [orfLengthCutoff] [orfReadCutoff]
    readCorrectedFile: input read mapping file after offset correction;
    candidateORFFile: candidate ORFs, genePred format;
    outputDir: output directory, with files reporting testing parameters and predicted translating probability;
    orfLengthCutoff [optional]: cutoff of ORF length (nt), default: 12;
    orfReadCutoff [optional]: cutoff of supported read numbe, default: 11. 

Example: perl ribORF.pl ./example.data/corrected.sample.mapping.sam ./example.data/candidate.ORF.txt ./example.data 10 10

For questions, please contact:
Zhe Ji (zhe_ji@hms.harvard.edu or zheji@broadinstitute.org)
