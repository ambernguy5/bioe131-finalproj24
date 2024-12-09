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
2. Subtype A: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/003/104/735/GCA_003104735.1_ASM310473v1/GCA_003104735.1_ASM310473v1_genomic.fna.gz
3. Subtype B: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/003/102/975/GCA_003102975.1_ASM310297v1/GCA_003102975.1_ASM310297v1_genomic.fna.gz
4. Subtype C: ^
5. Combinant Form CRF01_AE: ^
  
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
# 4. Download and process annotation tracks for each genome (GFF files) from NCBI database:
1. HIV Reference genome: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/864/765/GCF_000864765.1_ViralProj15476/GCF_000864765.1_ViralProj15476_genomic.gff.gz
2. Subtype A: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/003/104/735/GCA_003104735.1_ASM310473v1/GCA_003104735.1_ASM310473v1_genomic.gff.gz
3. Subtype B: https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/003/102/975/GCA_003102975.1_ASM310297v1/GCA_003102975.1_ASM310297v1_genomic.gff.gz
4. Subtype C: ^
5. Combinant Form CRF01_AE: ^

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

Load annotation track into jbrowse. If APACHE_ROOT doesn't work, repeat step 2 to find this correct path. Usually APACHE_ROOT='/var/www/html' for Linux setup.

```
jbrowse add-track HIVref.gff.gz --out $APACHE_ROOT/jbrowse2 --load copy --assemblyNames HIVref
```
Repeat this for all desired annotation tracks for each subtype, replacing the corresponding link, title.gff, and assemblyNames for different subtypes.

# 5. Multiple Sequence Alignment Viewer Plugin
Download the multiple sequence alignment file to be uploaded to the Msaview plugin
1. Go to https://www.ebi.ac.uk/jdispatcher/msa/clustalo
2. To view the MSA for a protein (Nef, Vif, Vpu, Env, Vpr) or the genome of subtypes A, B, C, and CRF01_AE copy one of the sequence clusters below and paste it into Clustal Omega where it says "Paste your sequence here".



3. Click "View Results", then "Result Files", then download the files labeled "alignment in FASTA format converted by Seqret"


Install the Msaview plugin on the JBrowse user interface 
1. On the Jbrowse interface, open the Tools dropdown menu at the screen's top left corner and click on plugin store.
2. In the plugin store, scroll to the Msaview by Colin Diesh and click install.
3. Open the add dropdown at the top left and click on the multiple sequence alignment view
4. Upload or enter the URL of the MSA file (stockholm or clustal format) of the sequence alignment.
6. Click open 

# 6. 3D Protein Viewer Plugin
Configure the 3D protein viewer plugin by editing your config.json file. This file is typically found in your $APACHE_ROOT/jbrowse2 or /var/www/html/jbrowse2 folder:
1. Copy and paste into your config.json file above assemblies
```
"plugins": [
  {
    "name": "Protein3d",
    "url": "https://unpkg.com/jbrowse-plugin-protein3d/dist/jbrowse-plugin-protein3d.umd.production.min.js"
  }
  ]
```
It should look something like this: ![image](https://github.com/user-attachments/assets/ce2b6daa-4fed-4e8b-bdc3-eadefd6a7da8)

1. On the Jbrowse interface, open the Tools dropdown menu at the screen's top left corner and click on plugin store.
2. In the plugin store, scroll to Protein 3d by Colin Diesh and click install.
3. When annotation track is open, hover over a protein, and right click. Click "Launch protein view"
4. When Protein view is up, click open file manually. Click PDB ID bubble and input the following PDB ID's provided (one at a time). Click Launch 3-D Protein Structure View:
   * 8FYJ-  Env BG505 SOSIP-HT2 bound to two CD4 
   * 6MEO - gp120 bound to CD4 and CCR5
   * 6HAK- RT bound to dsRNA
   * 6URI- Nef bound to CD4 and AP2
   * 7U0F- Rev bound tom tubulin ring
   * 6CYT- Tat bound to AFF4, P-TEFb, and TAR loop
   * 8FVJ- Vif bound to APOBEC3H, CBF-beta, ELOB, ELOC, and CUL5 dimer
   * 6XQJ: Vpr bound to hHR23A (NMR)
   * 4P6Z: Vpu bound to BST2 and AP1
5. Click the wrench icon to open settings.
6. Under download structure tab, reinput the PDB ID you first input. Click apply, which should take you to the State Tree Page
7. Click the Assembly 1 tab, then apply action, and finally 3D representation. Click apply.
8. Repeat steps for other proteins.
# 7. Synthesizing data for Feature annotation tracks
show how to convert tables to gff files
edit config.json to add assemblies and tracks
# 8. Linear Synteny and Dot Plot View



