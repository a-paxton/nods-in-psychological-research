# Introduction to Storrs HPC

This is a guide to using the Storrs High Performance Computing resources.
[The Storrs HPC Cluster guide](https://wiki.hpc.uconn.edu/index.php/Main_Page)
is the primary source of the UConn-specific information; I recommend checking
it out if you have further questions.

**Please note** that this is written by a Mac user, which means they should be
suitable for Mac and Linux users. I have provided some links to information for
Windows users where I have been able to find it.

***

## Connect to the cluster.
- If you are connecting from off-campus, connect to the VPN first.
- Open a new command line window, and use the following command:
  `ssh [your_netID]@login.storrs.hpc.uconn.edu`
    - Here and below, swap `[your_netID]` for your UConn-issued NetID.
- Enter your NetID password when prompted.
- *Windows users*: See [this page](https://wiki.hpc.uconn.edu/index.php/SSH_on_Windows)
  on the Storrs HPC Wiki for information about connecting.

## Get your bearings on your login node.
- Check out your current folder: `ls -la`
  - This will give you the contents of your current folder in a neat,
    easy-to-read format.
- Create a new folder for our tutorial: `mkdir psyc_5570`
- Move into the folder: `cd psyc_5570`

## Check out what's going on with the cluster.
- To check the status of the cluster, use the following command: `sinfo`
- To check the queue, use the following command: `squeue`
- To check your jobs in the queue, use: `squeue -u [your_netID]`
- To check out the modules available on the cluster, use: `module avail`
  - Search available modules by using: `module avail [term]`

## Launch an interactive session.
- Check out the available R versions: `module avail r`
- Load R: `module load r/3.1.1`
- Start an interactive session: `fisbatch --ntasks=24 --nodes=1 --exclusive`
- Launch R: `R`
  - If needed, install packages locally using `install.packages()`, as usual.
- Once finished with R, quit R: `quit()`
- Once finished with the interactive session, exit: `exit`

## Upload something from your local machine to your remote session.
- On your computer, create a new file called `test_script.R`
- In the file, add the following text and save the file:
  ```
  set.seed(100)
  write.csv(rand(1),'random_number.csv')
  ```
- In a new command line window, send your script to the cluster:
  `rsync -avzP /local/path/test_script.R [your_netID]@login.storrs.hpc.uconn.edu:~/psyc_5570`

## Try batching.
- On your computer, create a new file called `psyc_5570-test.sbatch`
- In the file, add the following text and save the file:
  ```
  #!/bin/bash

  #SBATCH --job-name=psyc_5570_test         # Name of the job (whatever you want)
  #SBATCH --output=slurm_output/psyc_5570_test.out       # Provide output file.
  #SBATCH --error=slurm_output/psyc_5570_test.err        # Provide error file.
  #SBATCH --array=1-5                   # Provide the number of times we want to run the file.
  #SBATCH --ntasks=1                    # We've only got one script we're running per job.
  #SBATCH --time=01:00:00               # Don't let it run forever

  # save our array ID as an environmental variable
  # export SLURM_ARRAY_TASK_ID

  # load the modules we need
  module load r/3.6.1

  # print to output for confirmation that it's started
  echo $SLURM_ARRAY": Running SLURM task"

  # run the program
  Rscript $HOME/psyc_5570/test_script.R

  # print to output for confirmation that it's ended
  echo $SLURM_ARRAY ": Job done"

  ```
- On your computer, alter the R script to define the following variable and
  use the variable to change the CSV filename:
  `task_id = Sys.getenv("SLURM_ARRAY_TASK_ID")`    
- Use `rsync` to move both your `.R` and `.sbatch` files to your login node.
- Next, make a file for the output: `mkdir slurm_output`
  - Stay in the `psyc_5570-test` directory; do not move into `slurm_output`.
- Now, let's run our job using: `sbatch psyc_5570-test.sbatch`
  - We can check the status using: `squeue -u [your_netID]`
- Once it's done, check your output.
  - Check the remote files in your command line: `ls -la`
  - If you did it correctly, you should be able to see a number of new CSVs with
    names that include the `task_id` variable as you specified it in your edited
    R file.
  - You can also check the errors and output by navigating to the `slurm_output/`
    directory that we created earlier. You can even use `rsync` to transfer them
    to your local machine for easier investigation. In a new terminal window, use:
    `rsync -avzP [your_netID]@login.storrs.hpc.uconn.edu:~/psyc_5570 /local/path/test_script.R `
