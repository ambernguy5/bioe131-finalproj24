# bioe131-finalproj24

This is the READme for the HIV genome browser.

# Instructions to setup JBrowse:

1.1: Install Node.js and HomeBrew using AWS free trial instance.

Node.js is a cross-platform JavaScript runtime environment that will make is easy to run JBrowse2 command-line tools.

First, check whether Node.js is already installed by running the following. If node v20 is already installed, you can skip to the next step.
```
node -v
```
If Node.js is not installed, install it according to your device instructions

Linux
For Linux, you can use the code below. See https://nodejs.org/en/download/package-manager for more detail.

Note: sudo, also known as "super user do", runs commands with root/admin privileges. This can cause harm to your machine if you run the wrong command! It is also, however, a critical tool when doing things like installs - if you try something and are denied due to permissions, sudo is often the solution.
```
# installs fnm (Fast Node Manager)
curl -fsSL https://fnm.vercel.app/install | bash
# activate fnm
source ~/.bashrc
# download and install Node.js
fnm use --install-if-missing 20
# verifies the right Node.js version is in the environment
node -v # should print `v20.18.0`
# verifies the right npm version is in the environment
npm -v # should print `10.8.2`
```
2.2. @jbrowse/cli
Run the following commands in your shell. This uses the Node.js package manager to download the latest stable version of the jbrowse command line tool, then prints out its version. This should work for both macOS and Linux.
```
sudo npm install -g @jbrowse/cli
jbrowse --version
```
If the sudo version doesn't run, then try: 
```
npm install -g @jbrowse/cli if the sudo version doesn't run.
```

2.3. System dependencies
Install wget, apache2, samtools, and tabix.

wget is a tool for retrieving files over widely-used Internet protocols like HTTP and FTP.

apache2 allows you to run a web server on your machine.

samtools and tabix, as we have learned earlier in the course, are tools for processing and indexing genome and genome annotation files.


Linux
```
sudo apt install wget apache2
brew install samtools htslib
```
3. Apache server setup
3.1. Start the apache2 server
AWS will have a public IP address that you need to identify in the aws_instructions.

Linux
```
sudo service apache2 start
```
3.2. Fetch auto-assigned IP address in your AWS instance summary page. Your web server can be accessed at http://ipaddress. 

3.3. Access the web server
Open a browser and type the appropriate url into the address bar. You should then get to a page that says "It works!" (for AWS there may be some additional info). If you have trouble accessing the server, you can try checking your firewall settings and disabling any VPNs or proxies to make sure traffic to localhost is allowed.

3.4. Verify apache2 server folder
For a normal linux installation, the folder should be /var/www or /var/www/html
Verify that one of these folders exists (it should currently be empty, except possibly for an index file, but we will now populate it with JBrowse 2). If you have e.g. a www folder with no www/html folder, and your web server is showing the "It works!" message, you can assume that the www one is the root directory.

Take note of what the folder is, and use the command below to store it as a command-line variable. We can reference this variable in the rest of our code, to save on typing. You will need to re-run the export if you restart your terminal session!

**Be sure to replace the path with your actual true path!**
```
export APACHE_ROOT='/path/to/rootdir'
```

3.5. Download JBrowse 2
First create a temporary working directory as a staging area. You can use any folder you want, but moving forward we are assuming you created ~/tmp in your home folder.
```
mkdir ∼/tmp
cd ∼/tmp
```
Next, download and copy over JBrowse 2 into the apache2 root dir, setting the owner to the current user with chown and printing out the version number. This version doesn't have to match the command-line jbrowse version, but it should be a version that makes sense.
```
jbrowse create output_folder
sudo mv output_folder $APACHE_ROOT/jbrowse2
sudo chown -R $(whoami) $APACHE_ROOT/jbrowse2
```
3.6. Test your jbrowse install
In your browser, now type in http://yourhost/jbrowse2/, where yourhost is either localhost or the IP address from earlier. Now you should see the words "It worked!" with a green box underneath saying "JBrowse 2 is installed." with some additional details.

4. Download and process HIV Genomic data FASTA files from NCBI database:
```
wget $FASTA_ROOT/dna/Homo_sapiens.GRCh38.dna_sm.primary_assembly.fa.gz
```
Unzip the gzipped reference genome, rename it, and index it. This will allow jbrowse to rapidly access any part of the reference just by coordinate.
```
gunzip Homo_sapiens.GRCh38.dna_sm.primary_assembly.fa.gz
mv Homo_sapiens.GRCh38.dna_sm.primary_assembly.fa hg38.fa
samtools faidx hg38.fa
```
Load genome into jbrowse
```
jbrowse add-assembly hg38.fa --out $APACHE_ROOT/jbrowse2 --load copy
```
