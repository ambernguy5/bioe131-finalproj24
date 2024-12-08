# bioe131-finalproj24

This is the READme to recreate the HIV genome browser. This READMe assumes the user already has apache2 and jbrowse-cli installed

#  1. Download system dependencies
Install wget, apache2, samtools, and tabix.

  1. wget is a tool for retrieving files over widely-used Internet protocols like HTTP and FTP.
  2. apache2 allows you to run a web server on your machine.
  3. samtools and tabix, as we have learned earlier in the course, are tools for processing and indexing genome and genome annotation files.

Linux
```
sudo apt install wget apache2
brew install samtools htslib
```
# 2. Verify apache2 server folder
For a normal linux installation, the folder should be /var/www or /var/www/html
Verify that one of these folders exists (it should currently be empty, except possibly for an index file, but we will now populate it with JBrowse 2). If you have e.g. a www folder with no www/html folder, and your web server is showing the "It works!" message, you can assume that the www one is the root directory.

Take note of what the folder is, and use the command below to store it as a command-line variable. We can reference this variable in the rest of our code, to save on typing. You will need to re-run the export if you restart your terminal session!

**Be sure to replace the path with your actual true path!**
```
export APACHE_ROOT='/path/to/rootdir'
```
For an AWS Linux instance:
```
export APACHE_ROOT='/var/www/html'
```
NOTE: APACHE_ROOT variable will erase if you restart your terminal.

# 3. Download and process HIV Genomic data (FASTA files) from NCBI database:
1. HIV Reference genome: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/864/765/GCF_000864765.1_ViralProj15476/GCF_000864765.1_ViralProj15476_genomic.fna.gz
2. Subtype A: insert subtype A FASTA link address
3. Subtype B: ^
4. Subtype C: ^
5. Subtype D: ^
  
Download reference HIV genomes for each subtype. Replace the link with corresponding genome after wget command
```
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/864/765/GCF_000864765.1_ViralProj15476/GCF_000864765.1_ViralProj15476_genomic.fna.gz
```
Unzip the gzipped reference genome, rename it, and index it. This will allow jbrowse to rapidly access any part of the reference just by coordinate.
```
gunzip GCF_000864765.1_ViralProj15476_genomic.fna.gz
mv GCF_000864765.1_ViralProj15476_genomic.fna HIVref.fa
samtools faidx HIVref.fa
```
Load genome into jbrowse
```
jbrowse add-assembly HIVref.fa --out $APACHE_ROOT/jbrowse2 --load copy
```
# 5. Download and process annotation tracks for each genome (GFF files) from NCBI database:
1. HIV Reference genome: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/864/765/GCF_000864765.1_ViralProj15476/GCF_000864765.1_ViralProj15476_genomic.gff.gz
2. Subtype A: insert subtype A GFF link address
3. Subtype B: ^
4. Subtype C: ^
5. Subtype D: ^

Commands for uploading HIV reference genome annotation track:
```
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/864/765/GCF_000864765.1_ViralProj15476/GCF_000864765.1_ViralProj15476_genomic.gff.gz
gunzip GCF_000864765.1_ViralProj15476_genomic.gff.gz
```

Use jbrowse to sort the annotations. 
    1. jbrowse sort-gff sorts the GFF3 by refName (first column) and start position (fourth column), while making sure to preserve the header lines at the top of the file (which start with “#”). 
    2. Compress the GFF with bgzip (block gzip, which zips files into little blocks for rapid access), and index with tabix. 
    3. The tabix command outputs a file named HIVref.gff.gz.tbi in the same directory, and we then refer to “HIVref.gff.gz” as a “tabix indexed GFF3 file”.
    4. Change 'HIVref' title to HIV1_groupM_subtypeA, B, C each time you upload an annotation track

```
jbrowse sort-gff GCF_000864765.1_ViralProj15476_genomic.gff > HIV_ref.gff
bgzip HIV_ref.gff
tabix HIVref.gff.gz
```

Load annotation track into jbrowse. Reminder: if APACHE_ROOT doesn't work, repeat step 2 to find this correct path. Usually APACHE_ROOT='/var/www/html' for Linux setup.

```
jbrowse add-track HIVref.gff.gz --out $APACHE_ROOT/jbrowse2 --load copy --assemblyNames HIVref
```
Repeat this for all desired annotation tracks for each subtype, replacing the corresponding link, title.gff, and assemblyNames for different subtypes.

# 4. 3D Protein Viewer Plugin
Download the protein viewer plugin using User Interface on genome browser. *ADD instructions for downloading the 3D protein viewer.
