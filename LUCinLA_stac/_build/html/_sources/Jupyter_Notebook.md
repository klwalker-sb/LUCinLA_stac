# Using Jupyter Notebook to interact with data on cluster
================================================================================================================================

Jupyter notebooks provide a platform to create and share documents that contain live code, equations, visualizations, and text.
via Jupyter notebook, you can visualize and analyze data straight on the cluster without needing to tansfer files or install new code.

## Install Jupyter notebooks on cluster

Create a folder in your home directory to house your Jupyter notebooks (e.g. `mkdir ~/Jupyter`)
You can clone the LUCinSA_helper repo [here](https://github.com/klwalker-sb/LUCinSA_helpers) or you can make your own after connecting to Jupyter.

Install Jupyter Notebook into your virtual environment on the cluster:
```
source .nasaenv/bin/activate
pip install jupyter notebook
```

## Connect to cluster via ssh with tunnel to port for Jupyter

If using PuTTY:

| **To connect to Jupyter via PuTTY:**    |                         |                                          |
| :---------------------------------:   | :----------------------------------:   | :-------------------------------------: |
| ![](/Images/PuttyForJupyter1.jpg)     | ![](/Images/PuttyForJupyter2.jpg)      | ![](/Images/PuttyForJupyter3.jpg)       |
| **1)** on first PuTTY screen, enter SSH address to bula as normal  | **2)** click `SSH`, then `Tunnels` on menu on left. In "Source Port", enter port number (8889 here. Pick a different number to avoid conflicts) | **3)** add another Tunnel to bellows in the same way, with something like 222 in the "Source Port" and <user>@bellows.eri.ucsb.edu:22 in the destination port. (the last number should match your original local port from the first PuTTY screen). Click "Add". 
    
Go back to session and save session with name (e.g. `JupyterOnERI` here) for future use

* Click "Open" to open the connection and enter password as normal.
* Activate your virtual environment that has Jupyter installed on it (i.e: `source .nasaenv/bin/activate`)
* At next prompt, enter: `jupyter notebook --no-browser --port=<port>` (using the port number from step 2 of your SSH connection)
* Leave this window running and open a browser from your computer and enter into the address bar: `http://localhost:<port>/`
You should now see a Jupter tree of your home directory on the cluster. Navigate to the notebook you want to use. 

(you can create a permanant password by entering `jupyter notebook password` at the prompt after activating a virtual environment with Jupyter installed, before entering the )
