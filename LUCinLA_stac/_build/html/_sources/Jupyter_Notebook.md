# Using Jupyter Notebook to interact with data on cluster
================================================================================================================================

Jupyter notebooks provide a platform to create and share documents that contain live code, equations, visualizations, and text.
via Jupyter notebook, you can visualize and analyze data straight on the cluster without needing to tansfer files or install new code.
:::{warning} Jupyter Notebook allows you to run code directly on the cluster without running through SLURM. This means that SLURM cannot account for your memory use when allocating other jobs -- PLEASE BE MINDFUL OF THIS and do not run jobs > 1-2 cores via Jupyter.
:::

## Install Jupyter notebooks on cluster

From any node on the cluster (i.e. bellows/quebracho), create a folder in your home directory to house your Jupyter notebooks (e.g. `mkdir ~/Jupyter`)
You can clone the LUCinSA_helper repo [here](https://github.com/klwalker-sb/LUCinSA_helpers) or you can make your own after connecting to Jupyter.

Install Jupyter Notebook into your base environment on the cluster (this is optiona; it will allow you to switch between kernels 
within your notebook).
```
conda install -c conda-forge notebook
conda install -c conda-forge nb_conda_kernels
```
optional: install extentions -- see [here](https://towardsdatascience.com/jupyter-notebook-extensions-517fa69d2231) for more on Jupyter extentions
```
conda install -c conda-forge jupyter_contrib_nbextensions
```
Next install a jupyter kernel into each virutal environment that you plan to run in Jupyter 
```
conda activate venv.lucinla38_dl
conda install -c conda-forge notebook
conda install ipykernel
```
:::{warning} It is highly recommended that you run Jupyter from a virtual environment and not your base environment
:::

If you do not have a virtual environment, create one. e.g.:
```
conda create --name  venv.lucinla38_jupyter python=3.8 pip ipykernel notebook
```
See [here](Environment) for details on setting up your virtual environment.


## Connect to cluster via ssh with tunnel to port for Jupyter

You first need to **connect to the UCSB network through a VPN** (Pulse Secure). This is currently necessary even if you are on campus.

**If on a Mac:**
ssh into the remote terminal as you normally do (eg. `ssh [username]@bellows.grit.ucsb.edu`)
On the remote terminal, activate the environment through which you will run Jupyter (eg.: `conda activate venv.lucinla38_dl`)
If this is your first time using Jupyter, create your permanant Jupyter password with `jupyter notebook password`
Now open a notebook on a second port (that you will connect to in the next step): `jupyter notebook --no-browser --port=<port>` where \<port\> is any 4-digit number.
Back in your local terminal, connect to the new port by way of the remote terminal:
`ssh -L <port>:localhost:<port> <username>@bellows.eri.ucsb.edu`
Open a web browser and go to the `url: http://localhost:<port>` (remember that \<port\> is the smae 4-digit number throughout) 
After entering your password at the prompt, you should see the file tree from your home directory.

**If using PuTTY:**

| **To connect to Jupyter via PuTTY:**    |                         |        
| :---------------------------------:   | :----------------------------------:   |
| ![](/Images/PuttyForJupyter1b.jpg)     | ![](/Images/PuttyForJupyter2b.jpg)      |
| **1)** on first PuTTY screen, enter SSH address to node as normal  | **2)** click `SSH`, then `Tunnels` on menu on left. In "Source Port", enter port number (8887 here. Pick a different number to avoid conflicts) |

:::{note} To use jupyter on quebracho, just replace `bellows` with `quebracho` in the first ssh command
:::

:::{note} `jupyter notebook` can be replaced with `jupyter lab` throughout
:::

:::{note} if your jupyter notebook uses interactive web tools (like leaflet, folium, etc.), you will need to open up a second port the same way you open the first (8887 here) to pass within your jupyter scripts
:::
    
Go back to session and save session with name (e.g. `JupyterOnERI` here) for future use

* Click "Open" to open the connection and enter password as normal.
* Activate your virtual environment that has Jupyter installed on it (i.e: `conda activate venv.lucinla38_jupyter`)
* At next prompt, enter: `jupyter notebook --no-browser --port=<port>` (using the port number from step 2 of your SSH connection)
* Leave this window running and open a browser from your computer and enter into the address bar: `http://localhost:<port>/`
You should now see a Jupter tree of your home directory on the cluster. Navigate to the notebook you want to use. 

(you can create a permanant password by entering `jupyter notebook password` at the prompt after activating a virtual environment with Jupyter installed, before entering the notebook)
