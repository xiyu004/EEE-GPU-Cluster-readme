# EEE-GPU-Cluster-readme
This is a guideline on how to use Slurm-based GPU cluster. You are to closely follow the examples listed here to use EEE's GPU cluster. There is no lines in the readme that you can skip. Read carefully!

To start with, you need to understand the following workflow for a Slurm-based GPU cluster:
- Login to the login node using ssh.
- Configure your coding environment under your home directory.
- Request GPU instances using command `srun` and `sbatch`.
- Available GPU node(s) will be assigned to you tempoarily and you may run your code on them.
- Once job finished, you can look for results in relevant files that saves the program's output.

You are highly recommanded to ask any AI chatbot such as ChatGPT for more information on Slurm's usage because this readme cannot inclose all aspects of Slurm's usage.

# SLURM Job Submission Guide

This guide explains how to use SLURM for job submission with `srun`, `sbatch`, and interactive jobs. Examples are included to help you understand each command's usage and functionality.

---

## Table of Contents
- [Introduction](#introduction)
- [Interactive Jobs](#interactive-jobs)
- [srun Command](#srun-command)
- [sbatch Command](#sbatch-command)
- [Command Explanation](#command-explanation)
- [Important Notes](#important-notes)
- [Debugging (Advanced)](#debugging-advanced)
- [References](#references)

---

## Introduction
SLURM is a highly configurable workload manager used in high-performance computing environments. This guide demonstrates how to:
- Run jobs interactively.
- Submit batch jobs using `sbatch`.
- Use `srun` for immediate execution or within scripts.
- You are recommanded to use `sbatch` because it saves you time to repeat some procedures.

---

## Interactive Jobs
Interactive jobs allow you to request resources and directly execute commands in a live session. This is useful for debugging or running short-term processes.

### Command Example
```bash
srun --pty -p partition_name --time=2:00:00 --mem=8G --cpus-per-task=4 bash
```

### Explanation
- `--pty`: Enables an interactive session.
- `-p partition_name`: Specifies the partition (replace `partition_name` with your cluster's partition name).
- `--time=2:00:00`: Requests a maximum runtime of 2 hours.
- `--mem=8G`: Requests 8 GB of memory.
- `--cpus-per-task=4`: Requests 4 CPU cores.
- `bash`: Starts a Bash shell.
- If your requested config is more than what the cluster could offer, you will not be assigned any node. You are recommanded to request less than 24G of RAM and less than 8 CPU cores.

---

## srun Command
`srun` is used to run a single task or a set of tasks in parallel, either interactively or as part of a script. It is often used for testing before submitting a batch job.

### Command Example
```bash
srun -p partition_name --time=1:00:00 --ntasks=1 --cpus-per-task=4 --mem=16G python my_script.py
```

### Explanation
- `-p partition_name`: Specifies the partition.
- `--time=1:00:00`: Requests a runtime of 1 hour.
- `--ntasks=1`: Specifies a single task.
- `--cpus-per-task=4`: Allocates 4 CPU cores to the task.
- `--mem=16G`: Allocates 16 GB of memory.
- `python my_script.py`: Executes the Python script `my_script.py`.

---

## sbatch Command
`sbatch` is used to submit batch scripts for non-interactive execution. This is suitable for long-running or scheduled tasks.

### Batch Script Example
Create a file `job.slurm` with the following content:
```bash
#!/bin/bash
#SBATCH -p partition_name
#SBATCH --job-name=my_job
#SBATCH --output=output.log
#SBATCH --error=error.log
#SBATCH --time=24:00:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G

module load Miniforge3
source activate my_environment
python my_script.py
```

### Submit the Job
```bash
sbatch job.slurm
```

### Explanation
- `#!/bin/bash`: Specifies the script interpreter.
- `#SBATCH -p partition_name`: Specifies the partition.
- `#SBATCH --job-name=my_job`: Names the job `my_job`.
- `#SBATCH --output=output.log`: Specifies the file for standard output.
- `#SBATCH --error=error.log`: Specifies the file for error messages.
- `#SBATCH --time=24:00:00`: Requests a maximum runtime of 24 hours.
- `#SBATCH --ntasks=1`: Specifies a single task.
- `#SBATCH --cpus-per-task=8`: Allocates 8 CPU cores to the task.
- `#SBATCH --mem=32G`: Allocates 32 GB of memory.
- The script then loads the `Miniforge3` module, activates a Conda environment, and runs the Python script `my_script.py`.

---

## Command Explanation

### Common SLURM Options
| Option               | Description                                      |
|----------------------|--------------------------------------------------|
| `-p` or `--partition`| Specifies the partition name.                   |
| `--time`             | Sets the maximum runtime for the job.           |
| `--mem`              | Requests memory per node or per task.           |
| `--cpus-per-task`    | Allocates CPU cores per task.                   |
| `--ntasks`           | Specifies the number of tasks.                  |
| `--output`           | Redirects standard output to a file.            |
| `--error`            | Redirects error messages to a file.             |
| `--pty`              | Enables an interactive session (used with `srun`).|

### Viewing Job Status
- To view your submitted jobs:
  ```bash
  squeue -u your_username
  ```
- To cancel a job:
  ```bash
  scancel JOBID
  ```
---
## Important Notes
If you have big dataset/ckpt that will be loaded during your training, you are recommandded to manually move the files to /tmp/ on the node assigned to you. While your home directory, i.e., ~/, is network mounted, /tmp/ is not. /tmp/ is in fact a 1TB local SSD and it reads and writes much much faster than reading/writing from your home directory. The downside is that you need to specify the a particular node because /tmp/ is not shared via network, thus it is not consistent across the cluster. You are highly recommanded to do this data transfer yourself. If you are found constantly abusing the bandwidth of your home directory, we will send you warnings and will consider remove your account if you don't stop. Additionally, we will keep your files that is modified within 7 days in /tmp/. A automatically cleanup will be ran everyday to remove outdated files on all nodes.

---
## Debugging (Advanced)
Debugging via Slurm requires additional steps as follows:

## Port Forwarding for GPU Node Access

### **Step 1: Submit an Interactive Job**

In the first terminal, allocate resources for your job by running the following command:

```bash
srun -N 1 --ntasks-per-node=8 --gres=gpu:1 --time=0:30:00 --pty bash
```

- This job will run for **30 minutes** (for testing purposes). **Keep this window running** to maintain the allocation.

---

### **Step 2: Access the Allocated Node**

Once the job is allocated, open a **new terminal** and follow these steps:

1. **Check the allocated node name** by running:

    ```bash
    squeue
    ```

2. **SSH into the allocated GPU node**:

    ```bash
    ssh <node_name>
    ```

    Example:

    ```bash
    ssh lab001
    ```

---

### **Step 3: Set Up SSH Tunnel (Keep This Running)**

Now, open another **new terminal** on your local machine and set up an SSH tunnel to forward the port for accessing the GPU node. Run the following command:

```bash
ssh -L 2222:<node_name>:22 user_name@10.97.16.163
```

- Replace `<node_name>` with the name of the allocated GPU node (e.g., `lab001`).
- Replace `user_name` with your username.
- `10.97.16.163` is the IP address of the login node.

**Important**: **Keep this terminal running** to maintain the SSH tunnel and continue forwarding the port. If you close this terminal, the port forwarding will be stopped, and you will lose the connection to the GPU node.

---

### **Step 4: Access the GPU Node from Your Local Machine**

In another **new terminal** on your local machine, use the following command to access the GPU node through the SSH tunnel:

```bash
ssh -p 2222 user_name@localhost
```

- `-p 2222`: Specifies the local forwarded port (2222).
- `localhost`: Refers to your local machine, but because of the SSH tunnel set up in Step 3, this will redirect the connection to the allocated GPU node.

### **Step 5: Connect your IDE to the GPU node**

Once you can ssh to this local port, you can proceed to let your IDE such as PyCharm and VSCode connect to this local forwarded port (It does not has to be 2222).  

If you are still unfamiliar with how to setup your IDE to debug remotely, you are advised to read their own documents. They should have a detailed guideline regarding how to configrue a remote python interpreter for debugging.  

---

## References
- [SLURM Documentation](https://slurm.schedmd.com/documentation.html)
- Contact your system administrator for partition and resource details.

---

This guide provides a starting point for running jobs on SLURM clusters. Adjust resource requests and configurations based on your specific use case and cluster setup.
