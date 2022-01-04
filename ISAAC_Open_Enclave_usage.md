## Description

These are some common usages of [ISAAC Open Enclave](https://oit.utk.edu/hpsc/isaac-open/) server at University of Tennessee. The usage of another cluster [ISAAC Next Gen](https://oit.utk.edu/hpsc/isaac-open-enclave-new-kpb/system-overview-cluster-at-kpb/) that uses SLURM scheduler is not included here.

## 1. Request an account on ISAAC [here](https://portal.acf.utk.edu/user_requests/new_user/welcome/)

UTK0011 is the default project for all users. If your group has a project on ISAAC, you can ask your PI to add you or you can make a request from this [website](https://portal.nics.utk.edu/accounts/login/), so you have access to the project directory. 

## 2. Login 
```console
ssh your_username@acf-login.acf.tennessee.edu
```
```console
pwd # Output is your home directory path. It should be similar to /nics/c/home/your_username
```
***NOTE: Your home directory only has 10 GB storage space, so DO NOT use your home directory for analysis, use the scratch directory (see below)***

## 3. Transfer files between ISAAC Open Enclave and your local computer 

There are several methods to transfer files between remote server (such as ISAAC Open Enclave) and your local computer. See more details [here](https://oit.utk.edu/hpsc/isaac-open/data-transfer/) 

* `scp` Do the following in your local computer terminal, do not need to log into the server.

```console
# from ISAAC to your local computer
scp -r your_username@datamover4.nics.utk.edu:THE_PATH_TO_THE_FILE_YOU_WANT_TO_TRANSFER ./Desktop  

# from your local computer to ISAAC
scp -r FILE_ON_YOUR_LOCAL_COMPUTER your_username@datamover4.nics.utk.edu:THE_PATH_OF_DESTINATION   
```

* Softwares, e.g. WinSCP for Windows and CYBERDUCK for Mac

  - Download [WinSCP](https://winscp.net/eng/download.php)
  - Choose SFTP for File Protocol, Host name: datamover4.nics.utk.edu; Port number: 22; User name: NetID, Password: password for NetID 

* Web-based Transfers: Globus (according to ISAAC website, Globus is the fastest and most efficient data transfer method available on the Open Enclave). 

## 4. Transfer files between clusters on ISAAC

Do the following when you are logged into ISAAC.
```console
# from Next Gen to Open Enclave
scp -r dtn1.isaac.utk.edu:/lustre/isaac/scratch/your_username /lustre/haven/user/your_username

# from Open Enclave to Next Gen
scp -r datamover4.nics.utk.edu:/lustre/haven/user/your_username /lustre/isaac/scratch/your_username 
```
## 5. Go to scratch directory

The scratch directory has unlimited storage space, so use the scratch directory to do your analysis. 
```console
cd $SCRATCHDIR

pwd # Output is your scratch directory path. It should be similar to /lustre/haven/user/your_username
```
***NOTE: From ISAAC WEB: Lustre scratch directories are NOT backed up. Files in scratch directories (only /lustre/haven/user/{username} directories) are deleted by the purging process if they have not been accessed or modified within 180 days. SCRATCH directory is purged every 60 days.***

***If you are done with your analysis, you should transfer the files to your local computer or copy them to your project directory.*** 

## 6. Go to project directory

If your group has a project, you can go to your project directory with `cd /lustre/haven/proj/project_number`. Project directory normally has 1 TB space, but your PI can purchase more storage space if needed. The project directory is **NOT** purged, so it is good for data backup.
 
## 7. Finding Modules

Some softwares are already installed on ISAAC, you can use them with `module load software`.
```console
# outputs all the module files available to load on the cluster
module avail

# to search for a specific one, such as trimmomatic
module avail trimmomatic

# load the software you want to use
module load trimmomatic/0.36

# unload the software
module unload trimmomatic/0.36
```
## 8. Submitting a job

The format of your scripts:
```console
#!/bin/sh
#PBS -S /bin/bash
#PBS -A ACF-UTK0011
#PBS -l qos=campus
#PBS -l partition=beacon
#PBS -l nodes=1:ppn=4,walltime=24:00:00
#PBS -m abe 
#PBS -M your_email_address_here
cd $PBS_O_WORKDIR
your_commands_here
```
You can use text editors, such as emacs, nano or vim, to edit your scripts. The following is an example using emacs. First you create the script file with emacs. 
```console
emacs create_directories.sh
```
Then you can copy and paste the following. DO NOT forget to change the email address section. You can use the up, down, left and right arrows on your keyboard to go to the location where you want to edit, e.g. email address. Press control + x + s to save your script file and control + z to quit the editor.
```
#!/bin/sh
#PBS -S /bin/bash
#PBS -A ACF-UTK0011
#PBS -l qos=campus
#PBS -l partition=beacon
#PBS -l nodes=1:ppn=4,walltime=24:00:00
#PBS -m abe 
#PBS -M your_email_address_here
cd $PBS_O_WORKDIR
mkdir project # create a directory called project
mkdir project/test # create a directory called test within the project directory
```
You do not need to submit a batch job for running simple commands like mkdir. The above is just an example to show you the format. After you saved your scripts, you can submit it with `qsub`. You will get output like this: 5492131.apollo-acf. 5492131 is the jobid
```console
qsub create_directories.sh 
```
## 9. Interactive job submission

When you log into ISAAC, you are on login node. You should only use login node for small tasks, such as scripts editing and job submission. **DO NOT RUN JOBS ON LOGIN NODE, BECAUSE YOU WILL SLOW EVERYBODY DOWN.** 

If you want to run jobs in the terminal (e.g. debugging) instead of submitting a job, you should submit a interactive job to request computing nodes first. Use `exit` command to complete the job and return to the login node when you are done.
```console
qsub -I -A ACF-UTK0011 -l partition=beacon -l nodes=1:ppn=8,walltime=24:00:00 
```
With long-utk qos, you can request maximum 72 hours walltime, the same as batch jobs
```console
qsub -I -A ACF-UTK0011 -l qos=long-utk -l partition=beacon -l nodes=1:ppn=8,walltime=72:00:00 
```
***NOTE: You will be back to your home directory when your interactive job is ready, just change to your working directory again.***

## 10. Specify which partition you would like to use

The default partitions are general, beacon and rho. If you do not specify which partition to use, the system will use the default ones.
```console
#PBS -l partition=beacon 
```
#### Available partitions

|                 Node Set                |     Intel® Xeon® CPU    | Nodes | Cores/Node | GB Mem/Node | Total Cores | Interconnect |
|:---------------------------------------:|:-----------------------:|:-----:|:----------:|:-----------:|:-----------:|:------------:|
| beacon                                  | E5-2670                 | 43    | 16         | 256         | 688         | FDR          |
| beacon-gpu                              | E5-2670 NVIDIA K20X GPU | 4     | 16         | 256         | 64          | FDR          |
| rho                                     | E5-2670                 | 34    | 16         | 32          | 544         | QDR          |
| sigma                                   | E5-2680 v3              | 101   | 24         | 128         | 2,424       | FDR          |
| sigma_bigcore                           | E5-2680 v4              | 10    | 28         | 128         | 280         | FDR          |
| skylake                                 | Gold 6148               | 73    | 40         | 192         | 2,920       | EDR          |
| Cascade Lake                            | Gold 6248 Gold 6248R    | 15 13 | 40 48      | 192 192     | 600 624     | EDR          |
| monster                                 | E5-2687W v4             | 1     | 24         | 1,024       | 24          | Ethernet     |
| Cascade Lake Big Memory (bigmem)        | Gold 6248R              | 4     | 48         | 1,536       | 192         | EDR          |
| rome                                    | AMD EPYC 7542           | 1     | 32         | 128         | 32          | EDR          |
| Cascade Lake with GPU (Coming Aug 2021) | Gold 6248R V100S GPU    | 4     | 48         | 192         | 192         | EDR          |
| Totals                                  |                         | 303   |            |             | 8,584       |              |

## 11. Walltime with [default project UTK0011](https://oit.utk.edu/hpsc/isaac-open/using-qos-private-condo/) 

Default QoS is campus which only allows 24 hours walltime. You can change it to long-utk for 72 hours walltime. If you have a condo project, you can specify qos as condo for 28 days walltime.
```console
#PBS -l qos=long-utk
```

| QoS Attribute (-l qos={value}) | Min. Allocation | Max. Allocation | Wall Clock Limit |
|:------------------------------:|:---------------:|:---------------:|:----------------:|
| condo                          | 1 Node          | Condo Max.      | 28 Days          |
| campus                         | 1 Node          | 24 Nodes        | 24 Hours         |
| overflow                       | 1 Node          | 24 Nodes        | 24 Hours         |
| long (UTHSC Projects Only)     | 1 Node          | 24 Nodes        | 6 Days           |
| long-utk (UTK Projects Only)   | 1 Node          | 18 Nodes        | 3 Days           |

## 12. Node-exclusive

If you want to run your jobs using the entire node rather than sharing it with other jobs
```console
#PBS -n
```
## 13. Job status
* Check the status of your submitted jobs
```console
qstat -a
```
* Check the status of your jobs in detail
```console
showq -w user=username
```
* Check the estimated starting time of your submitted job. When you submit a job, you get a jobid #, such as 5492131.apollo-acf
```console
showstart jobid(e.g. 5492131)
```
* Get all the details about a particular job (full status):
```console
qstat -f jobid(e.g. 5492131)
```
* Cancel a submitted job
```console
qdel jobid(e.g. 5492131)
```
## 14. Run dependency jobs
```console
# hold job1
JID1=`qsub -h job1`
# run job2 if job1 has terminated without error
JID2=`qsub -W depend=afterok:$JID1 job2` 
# run job3 if job2 has terminated without error
qsub -W depend=afterok:$JID2 job3 
# Release the first job to initiate
qrls $JID1 

# another option to run dependency jobs
# submit job1
qsub job1
# run job2 if job1 has terminated with error
qsub -W depend=afternotok:jobid_of_job1 job2
```
#### Parameters for running dependency jobs

| Parameters    | Description                                                          |
|---------------|----------------------------------------------------------------------|
|  after        | Execute current job after listed jobs have begun                     |
|  afterok      | Execute current job after listed jobs have terminated without error  |
|  afternotok   | Execute current job after listed jobs have terminated with an error  |
|  afterany     | Execute current job after listed jobs have terminated for any reason |
|  before       | Listed jobs can be run after current job begins execution            |
|  beforeok     | Listed jobs can be run after current job terminates without error    |
|  beforenotok  | Listed jobs can be run after current job terminates with an error    |
|  beforeany    | Listed jobs can be run after current job terminates for any reason   |