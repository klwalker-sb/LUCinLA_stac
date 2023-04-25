# quebracho
======================================================================================================================================
quebracho is a new fat node with GPU for primary use by our lab.

Hardware specs: 
* 64 CPU
* NDIVIA A40 GPU

Software specs:
* CUDA = 
* GDAL = 3.5.0
* RStudio (in progress)
* Zabbix for usage monitoring (in progress)

## To log in to Quebracho:

Contact Kendra Walker or Gavin McDonald to get added to the user list for quebracho
fill out your credentials [here](https://dc1.grit.ucsb.edu/)
see [here](ssh keys) for details on setting up SSH keys 

To SSH into the cluster, use:
```ssh <username>@quebracho.grit.ucsb.edu```
(if you had an account with ERI prior to quebracho, you might need to ssh into grit.ucsb.edu or eri.ucsb.edu first)  
see [here](connecting) for details on connecting to cluster environments from a Mac/linux or Windows environment

## Running jobs:

You will need to set up an environment and run jobs through the job manager: SLURM
See [here]((Environment) for general info on setting up an environment

Here is an example of a process to set up an environment to run the pytorch-based library, cultionet (a public git repo for field segmentation):

```
conda create --name .cnet38 python=3.8
conda activate .cnet38
conda config --add channels conda-forge
conda config --set channel_priority strict
conda install cython>=0.29.* numpy<=1.21.0
pip install torch==1.12.1+cu113 torchvision==0.13.1+cu113 torchaudio==0.12.1 --extra-index-url https://download.pytorch.org/whl/cu113 --no-cache-dir
python -c "import torch;print(torch.cuda.is_available())" # GPU check: True if cuda/GPU is available. if False, make sure you're not on bellows
pip install torch-scatter torch-sparse torch-cluster torch-spline-conv torch-geometric torch-geometric-temporal --extra-index-url https://data.pyg.org/whl/torch-1.12.1%2Bcu113.html --no-cache-dir
conda install -c conda-forge gdal
pip install cultionet@git+https://github.com/jgrss/cultionet.git
cultionet train -h #### cultionet install check. if errors, downgrade pytorch-lightning: pip install pytorch-lightning==1.4.0
```

For most jobs, you will need to submit through the queue via SLURM (this makes sure that we are not collectively making demands beyond our resources). We are installing Zabbix for additional monitoring capabilities for processes such as RStudio and Jupyter Notebooks that cannot be run through SLURM. The commands `htop` and `top` can also be used to monitor memory usage.

See [here](Downloading) for an example of a SLURM script.
See [here](SlurmCommands) for general SLURM commands. 
