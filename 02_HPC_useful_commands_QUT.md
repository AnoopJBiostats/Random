# 🖥️ QUT HPC: Useful Commands, Scripts, and Tips

This document includes a quick reference guide for submitting jobs, monitoring resources, using interactive jobs, setting up modules, and customizing your HPC environment via `.bashrc`.

---

## 1. SUBMITTING PBS JOBS
```bash
qsub job_script.sh
```

## 2. Monitor Job Queue (Live Updates)
```bash
watch qstat
qstat
qdel <job_id>
```

## 3. View Available Resources on HPC
```bash
qstat -q
```

## 4. Array Jobs
```bash
qstat -t <job_id>[].aqua # General Status (Running, Queued, Error)
qjobs -x <job_id>[].aqua # Resources Requested per Job
qstat -f <job_id>[].aqua # Detailed Status of Entire Array
qstat -f <job_id>[<index>].aqua # Detailed Status of a Specific Job in Array
```

## 5. Interactive Jobs
Lyra HPC Examples
```bash
# With GPU (A100)
qsub -I -S /bin/bash -l walltime=12:00:00,ncpus=5,mem=150GB,ngpus=1,gputype=A100 -N interactive_gpu_job

# Long CPU-only session
qsub -I -S /bin/bash -l walltime=95:00:00,ncpus=8,mem=100GB -N interactive_cpu_job

# Short CPU-only session
qsub -I -S /bin/bash -l walltime=12:00:00,ncpus=8,mem=50GB -N interactive_cpu_job_short
```

Aqua HPC Examples
```bash
# With queue specification
qsub -I -V -q cpu_inter_exec -l select=1:ncpus=8:mem=32GB -l walltime=8:00:00

# Without queue specification
qsub -I -V -l select=1:ncpus=8:mem=32GB -l walltime=8:00:00
```
PBS flags explained below
```bash
| Flag                  | Description                                |
| --------------------- | ------------------------------------------ |
| `qsub`                | Submits a job to the PBS scheduler         |
| `-I`                  | Starts an interactive job session          |
| `-V`                  | Exports your current environment variables |
| `-q cpu_inter_exec`   | Specifies the queue (for Aqua HPC)         |
| `-l select=1:ncpus=8` | Requests 1 node with 8 CPUs                |
| `mem=32GB`            | Requests 32 GB of RAM                      |
| `-l walltime=8:00:00` | Sets max time limit to 8 hours             |

```

## 6. Customizing your environment: .bashrc
The .bashrc file is sourced whenever a new shell session is opened. It’s essential for setting up your environment with modules, paths, aliases, or environment variables.
```bash
# Load modules on login
module load GCCcore/12.3.0
module load Python/3.11.3

# personalizing examples
alias la='ls -alh'
alias cd..='cd ..'

# intercative job - AQUA
alias qsu8="qsub -I -V -l select=1:ncpus=8:mem=32GB -l walltime=8:00:00 -N job8"

# Environmental variables - named settings that programs can read.
# PATH - tells Bash:“Where should I look for programs when you type a command?”
# It’s a list of folders separated by :
# If PATH = /usr/bin:/bin:/home/n11565608/.local/bin, then when you type python, Bash will search: /usr/bin/python, /bin/python, /home/n11565608/.local/bin/python
# ie, if you install programs in your home folder (not system-wide), you need to add that folder to PATH.

# 1) the below code Keep the old PATH ($PATH), Add /home/n11565608/.local/bin at the end. So now, Bash can also find programs in that folder
export PATH=$PATH:/home/n11565608/.local/bin

# 2) Custom tool paths
# ~/pgx_env/bin = folder where your PyPGx tool was installed.
# Putting it before $PATH means Bash will prefer this folder’s programs over system defaults.
# So if there’s both a python in /usr/bin and in ~/pgx_env/bin, the one in pgx_env/bin gets used first.
export PATH=~/pgx_env/bin:$PATH

# 3) Tool-specific variables
# PyPGx looks for this variable to know where your bundle files are stored.
export PYPGX_BUNDLE=~/pypgx-bundle
```

How to Apply Changes Immediately
```bash
source ~/.bashrc
```

## Copy files
this is helpful for bigger files
```bash
rsync -ah --progress /from-folder/ /to-folder/
```




