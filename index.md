<script> https://github.com/Mottie/GitHub-userscripts/blob/master/github-collapse-in-comment.user.js </script>

## Using UNIX and the Lunarc LSEN HPC system

The purpose of this workshop is to get you familiar with the UNIX environment and using the Lunarc LSENS system for high-performance computing. By the time you are done here, you should be able to do the following:

- Log into Luarc using the ThinLinc client to enter the Lunarc desktop environment
- Navigate the file structure and move files/folders around.
- Queue up a job to run on the compute nodes (a job for example could be a sequence alignment from an NGS run)
- More complicated bash usage with grep, awk, find, pipes and a tiny bit of bash scripting.

## Get your computer ready!

We need you to have installed the ThinLink client on the computer you bring to the workshop.
You can see this as a first exercise: Use google to learn about ThinLink, where to get it and how to install it for your system.  
Use this knowledge to install it ;-)

<details><summary>I can not use Google!</summary>
<p>
<a href="https://lunarc-documentation.readthedocs.io/en/latest/using_hpc_desktop/" target='_blank' >google: lunarc thinlink # first entry</a>
</p>
</details>

Security is a high priority an aurora and this starts with the server rejecting connections from all but the BMC network. So you can only use Aurora from within the BMC network.
TODO: SOMEONE ADD THE VPN INFO HERE?
 

### Connection info

- Server: aurora-ls2.lunarc.lu.se
- Username: your aurora username
- Password: your aurora password

One time password:
The one time password is created on your phone using either the 'Pocket Pass' app for MacOS or the '' app for android based phones.

## The ThinLink Interface

The window manager is based on [MATE Desktop](https://mate-desktop.org/).

Most scientific programs for Bioinformaticians come without a graphical user interface and therefore the 'Terminal' is our most basic, but likely most used tool.
The 'Terminal' is so important that the MATE desktop has a direcxt link to it in the top panel between the file explorer and the web browser icon.

In fact you could just click on that as we will use it extensively in this course.


### Data storage

All user data should be stored on our data server - not in your home folder!

```{bash, eval = FALSE  }
# home folder
ls ~
## data server (your folder) 
me=`whoami`
ls /projects/fs1/$me
```
<details><summary>More info on that</summary>
<p>
<ul>
<li> '~' is an inbuilt variable that points to your home directory.</li>
<li> 'me=`whoami`' creates a local variable with the output of the whoami (speak 'who am I') program which returns our username (stefanl for me) </li>
<li> ls /projects/fs1/$me simply lists the contents of the folder e.g. /projects/fs1/stefanl</li>
</ul>
</p>
</details>


### Create a symbolic link to the data folder in your home directory.

To allow this tutorial to work with one path to the data folder only please create a symbolic link to this folder in your home directory.

In the Terminal use the 'ln' program to create a link from '/projects/fs1/"your username"' to '~/NAS'.

<details><summary>How to create this link</summary>
<p>
This will make the data storage path available as ~/NAS</BR>

<pre><code class="bash">ln -s /projects/fs1/"your username" ~/NAS
</code></pre>
</p>
</details>


## Common programs:

Every Linux/Unix installation has a basic set of tools installed that are extremely helpful for the daily work.

Today we already used cd and ln which moves between directories respectively links files/directories to other locations.

Other very commonly used programs are ls, touch, cp, mkdir, mv, rm, echo, cat, cut, head, tail, grep, wc, tar, gzip, gunzip and <a href="https://www.tutorialspoint.com/unix_commands/index.htm", target='_blank'>others</a>.

You can read up on these programs using:

```{bash, eval = FALSE }
# get info to a program
man "program name"
```

Unfortunately this is not working on aurora. The internet also contains all this information.

### Create files / directories

Go to your data server folder, create a folder with the name 'TestFileCreation' and create a file named README.txt in the folder. The file should contain the string 'Here we play with files and folders'.

As a hint - you need the tools cd, mkdir and echo.

Just writing the program name without additional information into the Terminal gives help on how to use the program.

<details><summary>Solution</summary>
<p>
<pre><code class="bash">cd ~/NAS
mkdir TestFileCreation
cd TestFileCreation
echo 'Here we play with files and folders' > README.txt
</code></pre>
</p>
</details>

Now create a second folder and copy the README.txt  into this folder.
Add to the second file the line "The copied file has been modified" and compare that to the first file.


## Software installed on aurora

Aurora uses the [module system](https://www.nersc.gov/users/software/user-environment/modules/) to provide multiple versions of one program to all users.

Please add 'git' to your bash session using 'module load'.

<details><summary>Solution</summary>
<pre><code class="bash">module spider git # which git versions are there
module spider git/2.18.0 # what does git/2.18.0 depend on
module load GCCcore/7.3.0 git/2.18.0 # load the module
</code></pre>
</p>
</details>


## Usage of the compute nodes

The computer the ThinLink client connects you to is our frontend. ONE computer for all of you. So do not run computing intensive tasks there - NEVER. Instead you need to use our computing nodes to run heavy workload.

Aurora uses the [SLURM system](https://slurm.schedmd.com/) to manage the compute nodes.

Now please pull our test files from git using

```{bash, eval = FALSE }
cd ~/NAS
mkdir git
cd git
git clone git@gitlab:stefanlang/UnixCourseMaterial.git
cd UnixCourseMaterial
```

You can use 'find ./' to show all files and folder in this git repository.c
All our data is stored on a file server. This file server is mounted over a network connection which is significantly slower than the connections inside a computer. Hence it is extremely useful to store data that will be read multiple times, temporary files or not important outfiles on the compute node and copy the important outfiles back to the data server after the job has finished.

Lets first get information about the compute node harddisk configurations using 'df -h':

```{bash, eval = FALSE }
df -h
```

Prints the harddisk configuration of the frontend.

```{bash, eval = FALSE }
df -h > ~/NAS/TestFileCreation/Frontend_harddisks.txt
```

Stores this information into a file.

To run the same script in on the compute nodes we need a SLURM script that we can submit to the SLURM process manager. 

```{bash, eval = FALSE }
echo 'df -h > ~/NAS/TestFileCreation/Node_harddisks.txt' > ~/NAS/TestFileCreation/Query_NodeHarddisk.sh
```

But this file is not working - we need to modify it to look like that:

```
#! /bin/bash
#SBATCH -A lu2018-2-35 # the name of our aurora project
#SBATCH -n 1 # how many processor cores to use
#SBATCH -N 1 # how many processors to use (always use 1 here unless you know what you are doing)
#SBATCH -t 00:20:00 # kill the job after ths hh::mm::ss time
#SBATCH -J 'QnodeHD' # name of the job
#SBATCH -o 'QnodeHD_%j.out' # stdout log file
#SBATCH -e 'QnodeHD_%j.err' # stderr log file
#SBATCH -p lu # which partition to use
df -h > ~/NAS/TestFileCreation/Node_harddisks.txt #the real script part
```

Now try to run it using sbatch:

```{bash, eval = FALSE }
sbatch ~/NAS/TestFileCreation/Query_NodeHarddisk.sh
```

Can you see the difference?

How do we use the compute node local storage?
Lunarc has created a environment variable named $SNIC_TMP - lets see how to use that:

Add the line 'echo $SNIC_TMP >> ~/NAS/TestFileCreation/Node_harddisks.txt' to your script and run it again.

And now I want you to understand the difference:

First lets create a useless file of 94Mb

```{ bash, eval= FALSE }
dd if=/dev/zero of=~/NAS/TestFileCreation/file.txt count=1024 bs=100024
```

Add the lines
```
time cp ~/NAS/TestFileCreation/file.txt $SNIC_TMP
time cp $SNIC_TMP/file.txt $SNIC_TMP/localCopy.gff3
```

On the time of writing I got these values:

```
real	0m5.124s
user	0m0.000s
sys	0m0.119s

real	0m0.067s
user	0m0.002s
sys	0m0.065s
```

But where are they?


<details><summary>Solution</summary>
<p>
In the file QnodeHD_"batch job id".err in your working directory.
</p>
</details>



## More complicated Terminal usage


This part of the course is optional if we have enough time. If not you are welcome to try it on your own and drop by if there are questions.

### Extract information from a file

On aurora we store genome information in a special folder that is readable by all users:

```{bash, eval=FALSE }
ls /projects/fs1/common/genome/lunarc/
# e.g. gencode information mv19
ls /projects/fs1/common/genome/lunarc/genomes/mouse/GRCm38.p6/gencode.vM19.chr_patch_hapl_scaff.annotation.gtf
```

These folders soon get very long and therefore I recommend you to create links (again):

Link the /projects/fs1/common/genome/lunarc folder to your home directory.

<details><summary>How to create this link</summary>
<p>
This will make the data storage path available as ~/lunarc</BR>

<pre><code class="bash">ln -s /projects/fs1/common/genome/lunarc/ ~/lunarc
</code></pre>
</p>
</details>

Now get some information about the gencode.vM19.chr_patch_hapl_scaff.annotation.gtf file using ls, wc, head and tail.

Perfect - now you know how big these gtf files are and which internal structure they have. It would be quite interesting to get information out of the file - or?

Get all information regarding Gapdh out of the file (using grep) and put this information in a new 'Gapdh.gtf' file in your TestFileCreation folder. And as this is a step you might forget after some time I recommend you to store how you did that in the file 'Gapdh.gtf.log'.

<details><summary>How to ...</summary>
<p>
<pre><code class="bash">cd ~/NAS/TestFileCreation
grep Gapdh ~/lunarc/genomes/mouse/GRCm38.p6/gencode.vM19.chr_patch_hapl_scaff.annotation.gtf > Gapdh.gtf
echo 'grep Gapdh ~/lunarc/genomes/mouse/GRCm38.p6/gencode.vM19.chr_patch_hapl_scaff.annotation.gtf > Gapdh.gtf' > Gapdh.gtf.log
</code></pre>
</p>
</details>

<details><summary>For specialists please explain ;-)</summary>
<p>
<pre><code class="bash">history | tail -n2 | head -n1 | perl -lane 'shift(@F); print join(" ", @F);' > Gapdh.gtf.log
</code></pre>
</p>
</details>

## Pipes

Pipes are a Linux/Unix way to copy information between program calls. Pipes are used all the time in Bioinformatics and therefore the [Pipe concept needs explanation](https://en.wikipedia.org/wiki/Pipeline_(Unix)). 

You have used pipes all the time as each | and > is 'piping' information from one process to another or to a file.

I do not want to go into detail here, but read up on that it might become important for you later on.

## Extract with more conditions

If you would want to extract all genes from a certain chomosome area you need more than just one grep.
You could use awk here!

Task: get all genes from chr4 between bp 10000002 and 19000002 and store the info in the file 'chr4_10000002_19000002.genes.gtf'.

<details><summary>Use awk to get gene information</summary>
<p>
<pre><code class="bash">fname=~/lunarc/genomes/mouse/GRCm38.p6/gencode.vM19.chr_patch_hapl_scaff.annotation.gtf
grep -w gene $fname | awk '{ if ( $1 == "chr4" && ($4 < 19000002 && $5 > 10000002) ) {print}}' > chr4_10000002_19000002.genes.gtf
</code></pre>
</p>
</details>

Have you thought about the .log file? No - do that ;-)

I am sure you can think of more problems that can be solved like that.

## Find - a very useful tool! Likely skip VERY COMPLICATED

The Linux/Unix find program is extremely useful if you want to apply any of the before mentioned steps to a list of files/folders. The tool is very powerful and with power comes responsibility. Be sure that your scripts do what they should do - using test files.

Anyhow - 'find' does exactly that - it does find files for you:

```{bash , eval=FALSE }
find ~
```

It has a lot of options of which I use these: 
-name '*.R' finds all R scripts in the folder and the sub-folders
-type d finds all directories (f files)
-exec grep test {} + applies 'grep test' to all identified files
-maxdepth 1 stops at the first sub-folder level

As an exercise please get all genes from all installed mouse gtf files in the area chr4:10000002-19000002 and put them into separate outfiles.


<details><summary>Oops - that is complex...</summary>
<p>
<pre><code class="bash">for f in $(find ~/lunarc/genomes/mouse/ -name '*.gtf' ); do bname=basename $f
 awk '{ if ( ($3 ~ /gene/ && $1 == "chr4" ) && ($4 < 19000002 && $5 > 10000002 )) { print } }' "$f" > " ~/NAS/TestFileCreation/${bname}"; done
</code></pre>
Honestly this did take about 30 min for me, too ;-)
<a href="https://stackoverflow.com/questions/45880730/awk-get-information-on-input-and-output-filenames-from-file">Here</a> I learned that a bash for loop might be the easiest way to deal with all problems (in the last comment).</BR>
Problems here: How to split the result up into multiple outfiles?</BR>
Fix: temporary variable bname to be populated in the for loop bash line 1 and after that call awk in bash line 2.
The bash lines are separated by '\n' after the do statement of the for loop and terminated with a '; done'.
</p>
</details>





