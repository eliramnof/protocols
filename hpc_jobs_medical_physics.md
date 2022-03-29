
# Running jobs on the Medical Physics cluster (mp_clin1) using conda environments and LSF job scheduler. 

## Setting up remote workspace
1. map remote server to macOS Finder:

    `sshfs user@MP_clin1:/ ~/mount -ovolname=MP_clin1`
    
    Will be prompted for password

    *This is useful for easily transferring files between your local machine and the working directory on the cluster (without `scp`), as well as for using the explorer in a text editor (see more below).*
2. connect to remote server in terminal (also convenient to use a text editor such as [Visual Studio Code](https://code.visualstudio.com/) which has a built-in terminal window and folder management pane)

    `ssh user@mp_clin1`

    Will be prompted for password 

## Setting up for HPC use
3. Reserve HPC resources (e.g. below) using [LSF](https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=reference-command) : 

    `bsub -m "pllimphsing4" -R rusage[mem=128GB] -q research -n 2  -W 3:0000 \ -gpu "num=2:mode=exclusive_process:mps=no:j_exclusive=yes" -Is /bin/bash`

    - `-m` submits job to be run on a specific host, host group, or compute units (in the above example `pllimphsing4`). See available resources using `bhosts` and `bhosts -gpu`. 
    - `-R` runs the job on a host that meets the specified resource requirements (in this case memory, `rusage[mem=64GB]`. 
    - `-q` submits the job to one of the specified queues (e.g., `research`)
    - `-n` submits a parallel job and specifies the # of tasks in the job (e.g., `2`)
    - `-W` runtime limit of the job (e.g., `3:0000` is  3 hrs.)
    
        **make sure the time allocation is sufficient to finish the job**
    - `-gpu` specifies properties of gpu resources required (e.g., `"num=2:mode=exclusive_process:mps=no:j_exclusive=yes"` requests 2 GPUs)

    - `-Is` indicates an interactive ssh session
    - `/bin/bash` specifies a bash shell


## Conda environment
Create and activate conda environment (e.g., by using environment.yml file): 

`conda create env -f environment.yml`   
`conda activate <name of conda environment>`

- if you get an error about needing to initialize your shell or that the shell is not configured using `conda init` or `conda activate`, try restarting the conda shell via: 

    `source /nadeem_lab/miniconda3/etc/profile.d/conda.sh` 

    and try again. 

- **note:** your code may stop and complain about missing modules (especially PyTorch). In this case, try creating and activating the conda environment in the interactive bash shell from within the job (i.e., after submitting the `bsub` command). Notice that different nodes have different GPUs and this affects the conda environment build, so ideally build the conda environment where you will run the job. 

- **note:** if at any time you get an error about a missing home directory, log in directly to the specific node you are running and it will be created there automatically (e.g., `ssh user@pllimphsing4`)

## Using JupyterLab 
Jupyer notebooks are popular for evaluating results, variables, and mixing source code. [JupyterLab](https://jupyterlab.readthedocs.io/en/stable/getting_started/overview.html) is the next-generation user interface and offers additional features including an integrated debugger. 

- Reserve HPC resources and activate the conda environment (see above) and change directory to where the jupyter notebook is located (e.g., a jupyter notebook using Histocartography):

    `cd /nadeemlab/Eliram/Histocartography`

- To launch jupyter lab from the remote HPC server (with default port #): 
    `jupyter lab`

    (if jupyter lab isn't installed then install it using `pip install jupyterlab`)

    *note the port # output in the terminal or define using e.g., `--port=8080`

- In a new terminal access the notebook from your local machine over SSH and notice the name of the node being used is the same as the one reserved earlier using the `bsub` command: 

    `ssh -L 8888:localhost:8888 nofe@pllimphsing4`

- open browser on local machine and navigate to the localhost or using the url with token, e.g.,

    http://localhost:8888/labtoken=06f6b7160243a0f1a833554d4d0aa2f951958f2acab640a4


## HPC Resources at Medical Physics Cluster
There are 4 servers available (do not run things on the control node): 
CUDA Version: 10.2
| Node | # GPUs | Type of GPU |Memory per GPU|
|-----|-----|-------------|------|
|pllimphsing1  |4 |TITAN V|12,036 MB|
|pllimphsing2  |4 |TITAN V|12,036 MB|
|pllimphsing3  |4 |TITAN V|12,036 MB|
|pllimphsing4  |7 |GeForceRTX2080T|11,019 MB|
