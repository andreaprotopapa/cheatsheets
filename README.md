# CINECA GUIDE

1. [Getting started](#getting-started)
2. [Submit a job](#command-and-scripts-inside-the-cluster-cineca-to-submit-a-job)
3. [Additional infos](#cineca-additional-infos)
4. [Conda and git](#installations-of-conda-and-git-on-the-cluster)
5. [Singularity](#singularity-additional-infos)
6. [SLURM](#slurm-cheatsheet)

## Getting started 
For registration and account association follow:

https://wiki.u-gov.it/confluence/display/SCAIUS/UG2.1+Getting+started#expand-3Connectingtothecluster

**Update (08/09/2023)**: 
If you have already an account on CINECA, notice that it has recently change the authentication procedure for log-in into the cluster:
- follow this [guide](https://wiki.u-gov.it/confluence/display/SCAIUS/How+to+activate+the+2FA+and+configure+the+OTP) for activating the 2FA (send an email to superc@cineca.it to get the activation link)
- follow this [guide](https://wiki.u-gov.it/confluence/display/SCAIUS/UG2.1+Getting+started#expand-2Accountassociation) from point n.3, you will install [smallstep](https://smallstep.com/docs/step-cli/installation/#linux-packages-amd64) for creating a new certificate valid for 12 hours on your pc

## Command and scripts inside the cluster (CINECA) to submit a job
`./train.sh <num_cpu> <max_walltime>` (e.g. `./train.sh 12 24:00:00`)

train.sh:
```
	#!/bin/bash
	# >>> Pulling repos
	...
	sbatch --job-name=job_example --cpus-per-task=${1} --time=${2} --output=./slurm_output/job_example.out --error=./slurm_output/job_example.out train.sbatch
```

train.sbatch:
```
	#!/bin/bash
	#SBATCH --partition=g100_usr_prod
	#SBATCH --mem=20000M
	#SBATCH --ntasks=1
	#SBATCH --mail-type=ALL
	#SBATCH --mail-user=<your email>

	# >>> IF YOU NEED TO USE A CONTAINER FOLLOW THE CODE BELOW (otherwise use your code here)<<<
	# Load the module
	module load singularity
	# Run the container
	singularity exec --hostname ${SLURM_SUBMIT_HOST}${SLURM_JOB_ID} ./container.sif bash ./container_train.sh
```

container_train.sh:
```
	#!/bin/bash
	# >>> Activate the conda enviroment
	...
	# >>> Execute code
	...
```

## CINECA: Additional infos
https://wiki.u-gov.it/confluence/display/SCAIUS/UG3.3%3A+GALILEO100+UserGuide

Cineca allows the usage of TMUX as terminal multiplexer: https://tmuxcheatsheet.com/

Cineca works only offline inside the running node. Therefore: 
 - pull the repos before submitting the job (e.g. in `train.sh`)
 - to use a logger (e.g. [wandb](https://wandb.ai/site)):
 	- use `wandb_mode` as `offline`
 	- to sync with the server, inside the wandb folder: `wandb sync --include-offline ./offline-*`
	- Script for syncronize wandb offline runs (supponing to have a group directory containing more than one run)
      ```
		    #!/bin/bash

		    # argument 1: group directory
		    conda activate <env_name>

		    RAND_ID=$(python3 -c "import wandb; print(wandb.util.generate_id());")

		    echo "Syncing runs $1 to run new id $RAND_ID"

		    # first, sync last series of logs to new id
		    first_dir=$(ls -t $1| head -1)
		    wandb sync $1/$first_dir/wandb/$(ls -t $1/$first_dir/wandb/ | grep offline | head -1) --id $RAND_ID

		    # then all the others + last again to sync hyper-params.
		    for dir in $(ls $1)
		    do
        		run=$(ls $1/$dir/wandb/ | grep offline)
       			echo $run
       			wandb sync $1/$dir/wandb/$run --id $RAND_ID;
		    done
      ```
  
## Installations of conda and git on the cluster
- Install conda:
    ```
	    mkdir -p ~/miniconda3
	    wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
	    bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
	    rm -rf ~/miniconda3/miniconda.sh
	    ~/miniconda3/bin/conda init bash
	    ~/miniconda3/bin/conda init zsh
    ```
- Create a conda environment:
    ```
	    conda create -n soro_env python=3.8
	    conda activate <env_name>
    ```
- If singularity is not installed:
    ```
	    conda install -c conda-forge singularity
    ```
- Clone git repositories:
    ```
	    conda install gh --channel conda-forge
	    gh auth login
	    <gh token>
	    git clone <repo>
    ```


## Singularity: Additional infos

Usually a cluster (e.g. CINECA, HPC) do not allow the use of Docker for security reasons, however it is possible to use Singularity as alternative.
Singularity, differently from Docker, creates a container as a directory inside the original host filesystem. 
Therefore, if you have originally created the Docker container in the path ```/home/a/b/c```, Singularity would virtually create a path ```/home/a/b/c``` inside the actual host filesystem.
When you use Singularity for the first time you should be take note of these steps:
- add in  `~/.bashrc` file:
```
	  export SINGULARITY_CACHEDIR=/scratch/gpfs/$USER/SINGULARITY_CACHE
	  export SINGULARITY_TMPDIR=/tmp
 ```
- pull docker image `<docker_path>` and convert into a singularity image `<container>.sif` (in your login node)
 ```
	  module load singularity
	  singularity pull docker://<docker_path>
 ```
- NOTE: if you are not able to pull it from the cluster, you can copy a pre-existent .sif into the cluster
- test singularity container using
	```
      singularity shell <container>.sif
  ```
Useful links:
- https://www.hpc.iastate.edu/guides/containers
- https://researchcomputing.princeton.edu/support/knowledge-base/singularity
- https://docs.sylabs.io/guides/3.0/user-guide/build_a_container.html
- https://www.hpc.polito.it/docs/guide-slurm-it.pdf
- https://help.itc.rwth-aachen.de/en/service/rhr4fjjutttf/article/1f18ef48d8444f15bd908c592e0c44fb/
- https://docs.sylabs.io/guides/3.1/user-guide/cli/singularity_shell.html

## SLURM cheatsheet
- submit a job
	```sbatch <file_name>.sbatch```
- show all jobs 
	```squeue```
- show your jobs
	```squeue -u <username>```
- show job infos
	```scontrol show job <job_id>```
- partions status
	```sinfo```
- delete a job
	```scancel <job_id>```
- running an interactive session on a node
	```srun --nodes=1 --tasks-per-node=1 --pty /bin/bash```



