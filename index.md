
## The LSENS platform
The platform is composed of three parts:
- The login node (front-end)
- The blades (where computation is done)
- Data servers (where data is stored)

## Using UNIX and the Lunarc LSEN HPC system

The purpose of this workshop is to get you familiar with the UNIX environment and using the Lunarc LSENS system for high-performance computing. By the time you are done here, you should be able to do the following:

- Log into Lunarc using the ThinLinc client to enter the Lunarc desktop environment
- Navigate the file structure and move files/folders around.
- Queue up a job to run on the compute nodes (a job for example could be a sequence alignment from an NGS run)
- More complicated bash usage with grep, awk, find, pipes and a tiny bit of bash scripting.

## Get your computer ready!

Security is a high priority on Aurora and this starts with the server rejecting connections from all but the BMC network. This means you can only use Aurora from within the BMC network and the VPN you installed makes this possible (even from outside the BMC).

So fire up your VPN client and login.

## The ThinLink Interface

Open up Thinlinc and login using your credentials. The server details are:

- Server: aurora-ls2.lunarc.lu.se
- Username: your aurora username
- Password: your aurora password

One time password:
The login will prompt you for a one time passcode which is created on your phone using the 'Pocket Pass' app. Open the app and enter the 4 digit code that you set, and then enter the 6 digit number shown. You will then be logged into Aurora (the front-end). Repeat the process if it doesn't work.


You will find yourself in the Linux system, and the window manager is based on [MATE Desktop](https://mate-desktop.org/). It looks like most other operating systems with a menu etc. Take a moment to look around.

Most scientific programs for Bioinformaticians come without a graphical user interface (GUI) and therefore the 'Terminal' is our most used tool. The 'Terminal' is so important that the MATE desktop has a direct link to it in the top panel between the file explorer and the web browser icon.

Open a terminal.

### Basic UNIX file structure
To know where you are right now do:
```{bash, eval = FALSE}
pwd
```
To make a new folder do:
```{bash, eval = FALSE}
mkdir NewTestFolder
```
To make a new empty file do:
```{bash, eval = FALSE}
touch NewTestFile
```
To list all files and folders do:
```{bash, eval = FALSE}
ls
```
To copy the file you have made into the folder you made do:
```{bash, eval = FALSE}
cp NewTestFile NewTestFolder/NewTestFile_copy
```
To go into the new folder do:
```{bash, eval = FALSE}
cd NewTestFolder
```
or to use the full path:
```{bash, eval = FALSE}
cd /home/username/NewTestFolder
```
To rename a file use the move command `mv` (yes, its strange):
```{bash, eval = FALSE}
mv NewTestFile_copy NewTestFile_copy_version2
```
To actually move trhe file one level up do:
```{bash, eval = FALSE}
mv NewTestFile_copy_version2 ../
```
To go back one level and meet your file do:
```{bash, eval = FALSE}
cd ../
```
To delete the file you just moved do:
```{bash, eval = FALSE}
rm NewTestFile_copy_version2
```
To remove the folder you made do:
```{bash, eval = FALSE}
rm -R NewTestFolder
```

### Data storage
All user data should be stored on the data server - not in your home folder (your initial starting place when you login)! The home folder is on the front-end that doesn't have a lot of space, which is why big data must be kept on the data servers.

Your space on LS2 is kept at `/projects/fs1/username/no_backup` and you get there by doing:

```{bash, eval = FALSE  }
# home folder
ls ~
## data server (your folder) 
me=`whoami`
cd /projects/fs1/$me/no_backup
```

- **'~'** is an inbuilt variable that points to your home directory.
- **'me=`whoami`'** creates a local variable with the output of the whoami (speak 'who am I') program which returns our username (stefanl for me).
- **ls /projects/fs1/$me/no_backup** simply lists the contents of the folder e.g. /projects/fs1/stefanl


### Create a symbolic link to the data folder in your home directory.

Typing `/projects/fs1/$me/no_backup` everytime you want to go there is a pain, and we can get rid of this need by creating a symbolic link to your data folder in your home directory.

In the Terminal use the 'ln' command to create a link from '/projects/fs1/username/no_backup' to '~/NAS'. Do this by:

```{bash, eval = FALSE}
ln -s /projects/fs1/$me/no_backup ~/NAS
```

If you now type `ls` you will see the `NAS` folder there. You can now do:
```{bash, eval = FALSE}
cd NAS
```
Rather than having to do:
```{bash, eval = FALSE}
cd /projects/fs1/username/no_backup
```
Much nicer!

## Common programs

Every Linux/Unix installation has a basic set of tools installed that are extremely helpful for the daily work.

Today we already used cd and ln which moves between directories respectively links files/directories to other locations.

Other very commonly used programs are ls, touch, cp, mkdir, mv, rm, echo, cat, cut, head, tail, grep, wc, tar, gzip, gunzip and <a href="https://www.tutorialspoint.com/unix_commands/index.htm", target='_blank'>others</a>.

You can read up on these programs using:

```{bash, eval = FALSE }
# get info to a program
man "program name"
```

Unfortunately this is not working on aurora. The Internet also contains all this information.
All well programmed command line tools do also report a basic help if you use them in the wrong way - like without options at all. Try it:

```{bash, eval = FALSE}
mkdir
```

```{bash }
mkdir --help
```

### Create files / directories

Go to your data server folder (if you aren't already there):
```{bash, eval = FALSE}
cd ~/NAS
```
Create a folder with the name 'TestFileCreation':
```{bash, eval = FALSE}
mkdir TestFileCreation
```
Go into the folder:
```{bash, eval = FALSE}
cd TestFileCreation
```
Then create a file named README.txt in the folder. The file should contain the string 'Here we play with files and folders': 
```{bash, eval = FALSE}
echo 'Here we play with files and folders' > README.txt
```
To look at the contents of a file from terminal you can use the `more` command:
```{bash, eval = FALSE}
more README.txt
```

## Software installed on aurora

People use different versions of the same program, which means these programs cannot be loaded and ready to go. They need to be activated, and Aurora uses the [module system](https://www.nersc.gov/users/software/user-environment/modules/) to do this. For example, lets say that we want to use a program called `git`.

To add 'git' to your bash session we need to find the available versions:

```{bash, eval = FALSE}
module spider git
```
When we have decided which version we want we call `module spider` on the specific version to see what requirements it has:
```{bash, eval = FALSE}
module spider git/2.18.0
```
You will see that it also needs `GCCcore/7.3.0` (a dependency). To load these modules so you can use them do:
```{bash, eval = FALSE}
module load GCCcore/7.3.0 git/2.18.0
```
`git` will now be available to use for this session, and you can check this by calling:
```{bash, eval = FALSE}
git
```




<details><summary>Excercise: Try to load `samtools` version 1.6  into the session (we will use this later)</summary>
<pre><code class="bash">module spider samtools # shows which R versions are there
module spider SAMtools/1.6  # what samtools depends on
module load icc/2017.4.196-GCC-6.4.0-2.28  impi/2017.3.196 SAMtools/1.6
</code></pre>
</p>
</details>


## Usage of the compute nodes

The ThinLink client connects you to our front-end. This is ONE login computer for all of you, so do NOT run computing intensive tasks there - NEVER EVER. Instead you should use our blades to run compute heavy workloads.

Aurora uses the [SLURM system](https://slurm.schedmd.com/) to manage the job queue that distributes jobs to the  compute nodes

You have already loaded git, so use this to pull our test file which is an unsorted BAM file by using:

```{bash, eval = FALSE }
cd ~/NAS
mkdir git
cd git
git clone git@gitlab:stefanlang/UnixCourseMaterial.git
cd UnixCourseMaterial
```

Do:
```{bash, eval = FALSE }
ls
```
To see the files there. You will see `UnsortedTestFile.bam`.

### Sorting a BAM file using the blades

When certain jobs are run such as sequence alignment they create temporary files which are then removed/merged before the final output files are made. WE DO NOT WANT TEMPORARY FILES BEING WRITTEN TO THE STORAGE SERVERS! It creates unwantged and slow network traffic, but rather we want these files to be created on the blades instead which is much faster.

To send a job to the queue we need to make a SLURM script that tells the blades which programs to load and what to do on which files. In our example we are going to sort a small BAM file. To do this we will use the samtools program you learnt how to load earlier.

Firstly, open an editor and instert the following lines:


```
#! /bin/bash
#SBATCH -A lu2018-3-6 # the ID of our Aurora project
#SBATCH -n 1 # how many processor cores to use
#SBATCH -N 1 # how many processors to use (always use 1 here unless you know what you are doing)
#SBATCH -t 00:20:00 # kill the job after ths hh::mm::ss time
#SBATCH -J 'username_sort' # name of the job
#SBATCH -o 'username_sort%j.out' # stdout log file
#SBATCH -e 'username_sort%j.err' # stderr log file
#SBATCH -p dell # which partition to use
module load icc/2017.4.196-GCC-6.4.0-2.28  impi/2017.3.196 SAMtools/1.6
samtools sort -T $SNIC_TMP UnsortedTestFile.bam > SortedTestFile.bam 
```
Save this file as `sortbam.sh`. Now try to queue it up using sbatch:

```{bash, eval = FALSE }
sbatch sortbam.sh
```
You can see all the jobs on the queue using:
```{bash, eval = FALSE }
squeue
```

Your job will run, and the file SortedTestFile.bam will appear in time.

## What was `-T $SNIC_TMP` for?

Lunarc has created an environment variable named $SNIC_TMP which allows you to point out the temp folder to programs that can use one.


## More advanced terminal usage

This part of the course is optional if we have enough time. If we are out of time you are welcome to try it on your own and drop by if there are questions.


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

Get all information regarding Gapdh out of the file (using grep) and put this information in a new 'Gapdh.gtf' file in your TestFileCreation folder. And as this is a step you might forget after some time I recommend you to store the command that created the new gtf file in the file 'Gapdh.gtf.log'.

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
Do not try to understand that during the course. We do not have time for that.
<pre><code class="bash">history | tail -n2 | head -n1 | perl -lane 'shift(@F); print join(" ", @F);' > Gapdh.gtf.log
</code></pre>
</p>
</details>

## Pipes

Pipes are a Linux/Unix way to copy information between program calls. Pipes are used all the time in Bioinformatics and therefore the [Pipe concept needs explanation](https://en.wikipedia.org/wiki/Pipeline_(Unix)). 

You have used pipes all the time as each | and > is 'piping' information from one process to another or into a file.

I do not want to go into detail here, but read up on that it might become important for you later on.

## Extract with more conditions

If you would want to extract all genes from a certain chomosome area you need more than just one grep.
You could use awk here!

**Task:** get all genes from chr4 between bp 10000002 and 19000002 and store the info in the file 'chr4_10000002_19000002.genes.gtf'.

<details><summary>How to use awk to get gene information</summary>
<p>
<pre><code class="bash">fname=~/lunarc/genomes/mouse/GRCm38.p6/gencode.vM19.chr_patch_hapl_scaff.annotation.gtf
grep -w gene $fname | awk '{ if ( $1 == "chr4" && ($4 < 19000002 && $5 > 10000002) ) {print}}' > chr4_10000002_19000002.genes.gtf
</code></pre>
</p>

I created the fname variable to focus on the important parts of the awk call. You could of cause also get rid of the initial grep if you like.

</details>

You are of cause not limited to the usage of [awk](https://www.gnu.org/software/gawk/manual/gawk.html#Getting-Started) you could also implement the logics using Perl, Python or even C. 

In Perl the awk call would look like that:

```{bash, eval=FALSE }
perl -lane 'if ( $F[0] eq "chr4" and $F[2] eq "gene" and $F[3] < 19000002 and $F[4] > 10000002 ) { print } ' $fname > chr4_10000002_19000002.genes.gtf
```
It is a little slower in Perl than using awk, but Perl is not known for it's speed, rather for it's string manipulation capabilities and regular expressions.

Have you thought about the .log file? No - do that ;-)

I am sure you can think of more problems that can be solved like that.

## Find - a very useful tool!

The Linux/Unix find program is extremely useful if you want to apply any of the before mentioned steps to a list of files/folders. The tool is very powerful and with power comes responsibility. Be sure that your scripts do what they should do - using test files.

Anyhow - 'find' does exactly that - it does find files for you:

```{bash , eval=FALSE }
find ~
```

It has a lot of options of which I use these: 
- **-name '*.R'** finds all R scripts in the folder and the sub-folders
- **-type d** finds all directories (**f** files)
- **-exec grep test {} +** applies 'grep test' to all identified files
- **-maxdepth 1** stops at the first sub-folder level


## Advanced exercise - Likely skip VERY COMPLICATED

To make it very clear right at the start: I would not use a bash script to do this. Nevertheless I would recommend you to try this here and remember that you can do really complicated things using 'just' bash scripts.

BUT just because bash can handle the problem does not translate into one should use bash for the problem.
 
Please get all genes from all installed mouse gtf files in the area chr4:10000002-19000002 and put them into separate outfiles. And of cause do that in one go, not for each file separately. Do not hardcode the filenames.

This is a very complex task. Probably bad as an example here, but the more complex the more to learn.

Lets break this problem into smaller parts:
- **find** selects the files we want to work on
- **awk** selects the info we want to have
- **separate outfiles** This is a big problem that kept me thinking quite a lot

I figured that the easiest is to use [basename](https://linux.die.net/man/1/basename) and store this information in a [bash variable](http://tldp.org/HOWTO/Bash-Prog-Intro-HOWTO-5.html). After that we can use the variable to create infile specific outfiles.

The solution I found is using a [bash for loop](http://tldp.org/HOWTO/Bash-Prog-Intro-HOWTO-7.html) to iterate over all files found in the find call; store the basename of this file in a variable and use this variable to store the awk results and create the log file.

<details><summary>My solution</summary>
<p>
<pre><code class="bash">for f in $(find ~/lunarc/genomes/mouse/ -name '*.gtf' ); do bname=`basename $f`
 awk '{ if ( ($3 ~ /gene/ && ($1 == "chr4" || $1 == "4") ) && ($4 < 19000002 && $5 > 10000002 )) { print } }' $f > ~/NAS/TestFileCreation/${bname}
 history | tail -n2 | head -n1 | perl -lane 'shift(@F); print join(" ", @F);' > ~/NAS/TestFileCreation/$bname.log
 done
</code></pre>

Honestly it took me more than 30 min to find a first solution ;-).</BR>

<a href="https://stackoverflow.com/questions/45880730/awk-get-information-on-input-and-output-filenames-from-file">Here</a> I learned that a bash for loop might be the easiest way to deal with the "separate outfiles" problem (in the last comment).</BR>

Problem here: How to split the result up into multiple outfiles?</BR>

Fix: temporary variable bname to be populated in the for loop bash line 1. Right after that call awk in bash line 2.
The bash lines are separated by '\n' after the do statement of the for loop and terminated with a 'done' on the last line.</BR>

Do you find a way to get rid of the 'for loop'?
</p>
</details>


# You have made it!

Great! You have actually read and understand it all! But you still are untrained - use the Internet to get more informations - come and ask if you have a problem you can not solve on your own. 

And please keep on training!


## Additional info: SFTP - Getting data into and out of the secure area

You need an sftp client (FileZilla for example) in order to get data into or out of aurora-ls2.lunarc.lu.se.
You need to connect to port 22220 and you need your phone and the OTC application.

The IN folder is inbox/"your username>, the OUT folder outbox/"your username"
and you can only copy one file/folder at a time and not recursively. Therefore I recommend to tar whichever folder you want to send to lsens2 before you transfer it. Zip, bzip2 or gzip is of cause also possible.

