## Description

These are some common usages of [ISAAC Next Gen](https://oit.utk.edu/hpsc/isaac-open-enclave-new-kpb/system-overview-cluster-at-kpb/) (ISAAC-NG) at University of Tennessee. ISAAC-NG uses SLURM scheduler.

## 1. Request an account on ISAAC [here](https://portal.acf.utk.edu/user_requests/new_user/welcome/)

UTK0011 is the default project for all users. If your group has a project on ISAAC, you can ask your PI to add you or you can make a request from this [website](https://portal.nics.utk.edu/accounts/login/), so you have access to the project directory. 

## 2. Log into ISAAC-NG
```console
ssh your_username@login.isaac.tennessee.edu
```
```console
pwd # Output is your home directory path. It should be similar to /nfs/home/your_username
```
***NOTE: Your home directory only has 50GB storage space, so DO NOT use your home directory for analysis, use the scratch directory (see below)***

## 3. Transfer files between ISAAC-NG and your local computer 

There are several methods to transfer files between remote server and your local computer. See more details [here](https://oit.utk.edu/hpsc/isaac-open-enclave-new-kpb/data-transfer-new-cluster-kpb-2/) 

* `scp` Do the following in your local computer terminal, do not need to log into ISAAC.

```console
# from ISAAC to your local computer
scp -r your_username@dtn1.isaac.utk.edu:THE_PATH_TO_THE_FILE_YOU_WANT_TO_TRANSFER ./Desktop  

# from your local computer to ISAAC
scp -r FILE_ON_YOUR_LOCAL_COMPUTER your_username@dtn1.isaac.utk.edu:THE_PATH_OF_DESTINATION   
```

* Softwares, e.g. WinSCP for Windows and CYBERDUCK for Mac

  - Download [WinSCP](https://winscp.net/eng/download.php)
  - Choose SFTP for File Protocol, Host name: dtn1.isaac.utk.edu; Port number: 22; User name: NetID, Password: password for NetID 

* Web-based Transfers: Globus (according to ISAAC website, Globus is the fastest and most efficient data transfer method available on the ISAAC-NG cluster). 

## 4. Transfer files between clusters on ISAAC

Do the following when you are logged into ISAAC.
```console
# from Next Gen to Open Legacy
scp -r dtn1.isaac.utk.edu:/lustre/isaac/scratch/your_username /lustre/haven/user/your_username

# from Open Legacy to Next Gen
scp -r datamover4.nics.utk.edu:/lustre/haven/user/your_username /lustre/isaac/scratch/your_username 
```
## 5. Go to scratch directory

The scratch directory has 10 TB storage space, so use the scratch directory to do your analysis. 
```console
cd $SCRATCHDIR

pwd # Output is your scratch directory path. It should be similar to /lustre/isaac/scratch/your_username
```
***NOTE: Lustre scratch directories are NOT backed up and Not purged yet, but purging may begin at a later date. If you are done with your analysis, you should transfer the files to your local computer or copy them to your project directory. You are responsible for backing up your own data.***

## 6. Go to project directory

If your group has a project, you can go to your project directory with `cd /lustre/isaac/proj/project_number`. Project directory normally has 1 TB space, but your PI can purchase more storage space if needed. The project directory is **NOT** purged, so it is good for backing up your data.
 
## 7. Finding Modules

Some softwares are already installed on ISAAC-NG, you can use them with `module load software`.
```console
# outputs all the module files available to load on the cluster
module avail

# to search for a specific one, such as matlab
module avail matlab

# load the software you want to use
module load matlab/r2021b 

# unload the software
module unload matlab/r2021b 
```
## 8. Submitting a job

The format of your scripts. You can find detailed information [here](https://oit.utk.edu/hpsc/isaac-open-enclave-new-kpb/running-jobs-new-cluster-kpb/).
```console
#!/bin/bash
#SBATCH -J MyJobName            #The name of the job.
#SBATCH -A ACF-UTK0011            # The project account to be charged
#SBATCH --nodes=1                     # Number of nodes
#SBATCH --ntasks-per-node=48          # cpus per node 
#SBATCH --partition=campus            # If not specified then default is "campus"
#SBATCH --time=0-12:00:00             # Wall time (days-hh:mm:ss)
#SBATCH --error=MyJobName.e%J         # Errors will be written in this file
#SBATCH --output=MyJobName.o%J        # The output of the terminal will be written in this file
#SBATCH --qos=campus
#SBATCH --mail-user=your_email_address
#SBATCH --mail-type=ALL
your_commands_here
```
You can use text editors, such as emacs, nano or vim, to edit your scripts. The following is an example using emacs. First you create the script file with emacs. 
```console
emacs create_directories.sh
```
Then you can copy and paste the following. DO NOT forget to change the email address section. You can use the up, down, left and right arrows on your keyboard to go to the location where you want to edit, e.g. email address. Press control + x + s to save your script file and control + z to quit the editor.
```
#!/bin/bash
#SBATCH -J CreateDirectories
#SBATCH -A ACF-UTK0011
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=2
#SBATCH --partition=campus
#SBATCH --time=0-12:00:00
#SBATCH --error=CreateDirectories.e%J
#SBATCH --output=CreateDirectories.o%J
#SBATCH --mail-user=your_email_address
#SBATCH --mail-type=ALL
mkdir project # create a directory called project
mkdir project/test # create a directory called test within the project directory
```
You do not need to submit a batch job for running simple commands like mkdir. The above is just an example to show you the format. After you saved your scripts, you can submit it with `sbatch`. You will get output like this: Submitted batch job 9495. 9495 is the jobid.
```console
sbatch create_directories.sh 
```
## 9. Interactive job submission

When you log into ISAAC-NG, you are on the login node. You should only use login node for small tasks, such as scripts editing and job submission. **DO NOT RUN JOBS ON LOGIN NODE, BECAUSE YOU WILL SLOW EVERYBODY DOWN.** 

If you want to run jobs in the terminal (e.g. debugging) instead of submitting a job, you should submit a interactive job to request computing nodes first. 
```console
salloc -A ACF-UTK0011 --nodes=1 --ntasks=1 --partition=campus --time=01:00:00 --qos=campus # add --exclusive if you want exclusive access to nodes
```
## 10. Exclusive Access to Nodes

If you want to run your jobs using the entire node rather than sharing it with other jobs
```console
#SBATCH --exclusive
```
## 11. Job status

* Check the status of your submitted jobs
```console
squeue -u username
```
* Check the status of your job in detail
```console
scontrol show job jobid
```
* Cancel a submitted job
```console
scancel jobid(e.g. 9495)
```
* Check start time of your job
```console
squeue -start -j jobid
```
* Release/Hold a job
```console
scontrol release/hold jobid
```
* Modify the name of the job
```console
scontrol update JobID=jobid JobName=any_new_name
```
* Modify the total number of tasks
```console
scontrol update JobID=jobid NumTasks=Total_tasks
```
* Modify the number of CPUs per node
```console
scontrol update JobID=jobid MinCPUsNode=CPUs
```
* Modify the wall time of the job
```console
scontrol update JobID=jobid TimeLimit=day-hh:mm:ss
```

## 12. Run dependency jobs
```console
# submit job1.sh
sbatch job1.sh
# run job2.sh if job1 has terminated with error
sbatch --dependency=afternotok:jobid_of_job1 job2.sh
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