
(Environment)=
# Setting up your remote environment

## The general cluster environment

As with most cluster computers, the ERI system uses a UNIX-based shell called "BASH". This is a very simplified system that interprets code via command-line interface. To interact with the system, you need to use Shell scripting, which is based on UNIX/Linux command language. For a quick review of basic UNIX commands, [see here](Unix). For a more comprehensive coverage of working with BASH scripting, see these hpc-carpentry lessons on [navigating in bash](http://www.hpc-carpentry.org/hpc-shell/02-navigation/index.html) and [file manipulation in bash](http://www.hpc-carpentry.org/hpc-shell/03-files/index.html).

The point of high-performance computing is to manage and process large work loads. This is acheived through a work load manager, SLURM (Simple Linux Utility for Resource Management). Ultimate processing commands are sent to SLURM from .sh scripts in the bash directory via the command `sbatch`. Any other non-UNIX-based commands are processed within a virtual environment (e.g. via a Python interpreter).

Any processing done outside of SLURM (i.e. via IPython of Jupyter Notebook) can cause the whole system to crash for everyone because the resources used are not accounted for by the system when allocating to SLURM jobs. Do not run anything other than minimal tasks outside of SLURM.

## Setting up a virtual environment

Virtual environments allow us to customize our coding toolkit and preferences for each project within its own container so that it does not risk interfering with those of other projects or users. Within a virtual environment, you can define the version of Python that will be used, install packages specific to the project, and customize settings to facilitate your interactions, including aliases, which are shortcuts to replace frequently used or cumbersome commands.

For this project, we need Python 3.8 and the system we are using only has Python 3.7 installed. It is common for an HPC system to not have the exact version of Python desired for a project. There are various environmental managers that can be installed to solve this problem. [see here](managers) for a discussion of various environmental managers (Anaconda, Miniconda, Pyenv, Mamba) and the advantages and disadvantages of each. For the specific environment we are going to set up on this specific system, we have done the hard work of testing out all options for you and recommend using Miniconda.

After connecting to bellows via ssh for the first time, you will need to set up your environment.

Make sure you are in your home directory when creating/changing environment settings!
You should see your username \@bellows in the command prompt.
(there may be numbers in front; this is fine for now and can be taken care of later [here](profileIssue)).\
If you are unsure whether you are in your home directory, type `cd ~`

For this project, we will us Miniconda to create a virtual environment in Python 3.8 called venv.lucinsa38

### To install Miniconda:

On your own computer, go [here](https://docs.conda.io/en/latest/miniconda.html) and download `Miniconda3 Linux 64-bit`
Transfer this file into your home directory ([see here](Transfering) for ways to transfer files onto the cluster)
In your home dierctory on the cluster, type: `bash Miniconda3-<>.sh` (with <> filled in with the rest of the actual file name. 
(You can find the file name by typing `ls` from your home directory)).

Conda's base environment can be activated on startup by setting the activate_base parameter to true:
`conda config --set auto_activate_base true` (this may be the default)
If you prefer to not have it do this (i.e, if you use other environment managers as well), you can set this to false:
`conda config --set auto_activate_base false`
you will then need to type `conda activate` each time before you activate your specific environment

### To create your conda environment with specific version of python:

    conda create --name venv.lucinla38_dl python=3.8

***Activate your virtual environment:*** every time you run code that isn't BASH/SLURM from within the cluster:

    conda activate venv.lucinla38_dl

***Deactivate the virtual environment:*** (only when you are done!):

    conda deactivate

### Install Python tools/packages and dependencies from project github scripts:

To **interact with github on your virtual computer**, you first need to feed the remote computer the password. You can do this with [an osxcechain credential](https://docs.github.com/en/get-started/getting-started-with-git/updating-credentials-from-the-macos-keychain) or by simply caching it by entering the following into your local git bash or mac terminal:
`git config --global credential.helper cache`
Or in git for Windows enter: `git config --global credential.helper wincred`

**While in your home directory,** enter the following commands to setup your Python environment:

        # Activate the virtual environment
        conda activate venv.lucinla38_dl

        #Add conda-forge to your channels and make it a priority (this is key for this environment):
        (venv.lucinla38_dl) conda config --env --add channels conda-forge
        (venv.lucinla38_dl) conda config --env --set channel_priority strict

        #  install pre-reqs for geowombat (our main processing code)
        (venv.lucinla38_dl) conda install cython numpy=1.23.1 rasterio=1.3.0
        # install numpy AFTER installing gdal and make sure that both packages do not get altered
         (venv.lucinla38_dl) conda install gdal==3.5.0

        # install the main geowombat via Conda to let it solve the bulk of the environment (have patience, this can take up to 1hr)
        (venv.lucinla38_dl) conda install geowombat gdal=3.5.0 numpy=1.23.1 rasterio=1.3.0

        # Install eostac from github (this will also install the latest version of geowombat)
        (venv.lucinla38_dl) mkdir repos
        (venv.lucinla38_dl) cd repos
        (venv.lucinla38_dl) git clone -b KW_processing_db_ https://github.com/jgrss/eostac
        (venv.lucinla38_dl) cd eostac/
        (venv.lucinla38_dl) python setup.py build && pip install . gdal==3.5.0 numpy==1.23.1 rasterio==1.3.0

        # Install other repos needed for this project:
        (venv.lucinla38_dl) cd ../
        (venv.lucinla38_dl) git clone https://github.com/jgrss/geosample
        (venv.lucinla38_dl) cd geosample/
        (venv.lucinla38_dl) python setup.py build && pip install .
        
        (venv.lucinla38_dl) cd ../
        (venv.lucinla38_dl) git clone https://github.com/jgrss/stackstac
        (venv.lucinla38_dl) cd stackstac/
        (venv.lucinla38_dl) python setup.py build && pip install .

**to uninstall and reinstall a package from a github repo:** (i.e. if changes are made to the package that need to be incorporated)

        conda activate venv.lucinla38_dl
        (venv.lucinla38_dl) pip uninstall <package> -y
        (venv.lucinla38_dl) cd ~/repos/<package directory>
        #(look up the name of the main branch in the GitHub repo). Here it is 'main', but it could be 'master'/'dev'/etc. 
        (venv.lucinla38_dl) git pull origin main
        #Install new repo. If nothing else has changed it is safest to do this with --no-deps flag to avoid corrupting environment
        (venv.lucinla38_dl) python setup.py build && pip install --no-deps .
        #If the updated repo has new dependents that you need, install them individually (trying conda before pip), or if needed use:
        (venv.lucinla38_dl) python setup.py build && pip install . gdal==3.5.2 numpy==1.23.1 rasterio==1.3.0
        (venv.lucinla38_dl) conda deactivate

(configVim)=

### Configure vim (optional)

Text documents are edited in vim, which is not the most intuitive tool to those unfamiliar with it. Here you can find commands and tips for [working with vim](vimCommands). You can improve your experience with vim by creating a personalized
`.vimrc` file in your HOME directory ((output -\> \~/.vimrc) to configure vim.

Here is an example of a `.vimrc` doc (you can clone this from /raid-cel/sandbox/sandbox-cel/templates/home_vimrc.sh):

        # Add line numbers
        set number

        au BufNewFile,BufRead *.py
            \ set tabstop=4 |
            \ set softtabstop=4 |
            \ set shiftwidth=4 |
            \ set textwidth=79 |
            \ set expandtab |
            \ set autoindent |
            \ set fileformat=unix
        	
        highlight BadWhitespace ctermbg=red guibg=darkred
        au BufRead,BufNewFile *.py,*.pyw,*.pyx,*.pxd,*.c,*.h match BadWhitespace /\s\+$/

        set encoding=utf-8

        let python_highlight_all=1
        syntax on

        set cursorline

        set foldmethod=indent
        set foldlevel=99

        set ic

### Add a personalized .profile file (optional) {#add-a-personalized-profile-file-optional}

You can add a personalized `.profile` file in your HOME directory (output -\> \~/.profile).
This is useful if you want to create keyboard shortcuts (alias) or ensure global variables are properly set.

Here is an example of things you might want in a `.profile` doc (you can clone this from /raid-cel/sandbox/sandbox-cel/templates/home_profile.sh):

        # color enhancement for the terminal window
        export PS1="\[\033[36m\]\u\[\033[m\]@\[\033[32m\]\h:\[\033[33;1m\]\w\[\033[m\]\$ "
        export CLICOLOR=1
        export LSCOLORS=ExFxBxDxCxegedabagacad

        #simpler file navigation:
        alias .1='cd ../'
        alias .2='cd ../..'
        alias .3='cd ../../..'

        # shortcut to list SLURM job status ([username] = your username)
        alias qs="squeue -u [username]"

        if [ -n "$BASH_VERSION" ]; then
          if [ -f "$HOME/.bashrc" ]; then
            . "$HOME/.bashrc"
          fi
        fi

        if [ -d "$HOME/bin" ] ; then
          PATH="$HOME/bin:$PATH"
        fi

(profileIssue)=
\#\#\# Sourcing the profile (optional; only do if you created a .profile above)

:::{warning}Your personalized profile may not function correctly due to an irregular issue in the bash_profile.:::
:::

You could fix this everytime you log in by sourcing the profile:

    source .profile

But better to make the adjustment permanant by adding a`.bash_profile` to your home directory (output -\> \~/.bash_profile) with the following language (you can clone this from /raid-cel/sandbox/sandbox-cel/templates/home_bash_profile.sh):

    # .bash_profile
    # Get the aliases and functions
    if [ -f ~/.bashrc ]; then
      . ~/.bashrc
      fi

    if [ -f ~/.profile ]; then 
      . ~/.profile
    fi

    # User specific environment and startup programs

    PATH=$PATH:$HOME/bin
    BASH_ENV=$HOME/.bashrc

    #export USERNAME BASH_ENV PATH
    export TMOUT=0
:::