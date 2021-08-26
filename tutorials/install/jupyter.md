---
title: Install Jupyter Notebook
description: 
published: true
date: 2020-11-13T17:13:33.313Z
tags: 
editor: markdown
dateCreated: 2020-08-27T14:42:14.396Z
---

# Getting Started with and Installing Conda and Jupyter Notebook
 
## 1.) Installing conda (Anaconda3) on your workstation.

Use the following links to download and install conda (Anaconda3).

Download Conda (Scroll to bottom of site): https://www.anaconda.com/products/individual
Install Conda: https://docs.conda.io/projects/conda/en/latest/user-guide/install/linux.html

>Suggestion: If you don't already have a software directory in your home directory you should make one, then install anaconda there. When you install more software just keep it all in the same place.
{.is-info}

## 2.) Conda install Jupyter, MDAnalysis, and anything else your heart desires...

Now that you have anaconda3 use the following link to conda install jupyter lab and notebook. 

Jupyter Installation Instructions: https://jupyter.org/install

While you are at it install MDAnalysis: `conda install -c conda-forge mdanalysis`

>Suggestion: If you are on a rotation account type 'which conda' into the command line and see if conda is already installed. If it is you can choose to install your own version of Anaconda3 or use the one that is already installed. In order to conda install stuff using your version  of Anaconda3 make sure conda is set with the right path.
{.is-info}

The conda command is located in: `~/Software/anaconda3/bin/conda`

## 3.) Run your jupyter notebook locally.

Open jupyter notebook from command line using the following command.

`jupyter-notebook` 
or
`jupyter notebook`

A jupyter-notebook should automatically come up. If not, you might need to open a browser window and enter 'localhost:8888', the number following localhost might change. You will see what number to enter after you open a jupyter notebook from the commmand line. 

## 4.) Run your jupyter notebook remotely.

To start, ssh into your machine remotely.

Now enter the following into the command line. 

`jupyter-notebook --no-browser --port=8888`

This will get the jupyter notebook started on your workstation. Now in a new terminal on your local machine enter the following into the command line. 

`ssh -N -f -L localhost:8888:localhost:8888 username@workstation`

Next open your browser and enter the following.

`localhost:8888` 

Now you should be able to access your jupyter notebooks.

>Suggestion: When you are on your workstation remotely and want to open jupyter notebooks start this process in the parent directory. When you open jupyter you will find that you can only move down the directory tree from where you opened it. 
{.is-info}

I learned how to do this from here: https://amber-md.github.io/pytraj/latest/tutorials/remote_jupyter_notebook

You can save some typing by creating a **shortcut** in either the ~/.bashrc or ~/.bash_aliases file:

On your workstation:
```
 function jpt(){
     # Fires-up a Jupyter notebook by supplying a specific port
     jupyter notebook --no-browser --port=$1
}
```



On your local machine:
```
 function jptt(){
     # Forwards port $1 into port $2 and listens to it
     ssh -N -f -L localhost:$2:localhost:$1 username@workstation.umaryland.edu
}
```

After sourcing the ~/.bashrc or ~/.bash_aliases file, you can just type:

On your workstation:
`jpt 8000` (assuming you want to use port 8000)

On your local machine:
`jptt 8000 8000` (assuming you want to forward your notebook to port 8000)

The port number doesn't matter much. Generally anything > 8000 is not usually in use.

## 5.) Helpful add-on: Jupyter notebook extensions (optional but recommended)
To install: 
`conda install -c conda-forge jupyter_contrib_nbextensions`

Features I find most useful: 
- Code folding
- Execute time
- Table of contents
- Move selected cells

You can enable these extensions by using the jupyter subcommand. I find it easier to use the jupyter_nbextensions_configurator server extension. Once you open a jupyter notebook, you can find the **nbextensions config** under the **edit** menu.

You can read more about the extension here: https://github.com/ipython-contrib/jupyter_contrib_nbextensions

## 6.) Having issues? Maybe your localhost is already in use.

In order to check if the local host you are using is already in use type the following command. 

`lsof -ti:8888` 

If a number comes up then the localhost is in use. You can kill the localhost with the following command 

`lsof -ti:8888 | xargs kill -9` 

## 7.) Tired of Copying/Pasting Code Between Notebooks?

Common code between notebooks is well...common. But it can be cumbersome to find and also make your notebooks just look overwhelming and messy. 

Since jupyter notebookes rely on `ipython` as interface it inherently uses the start up functions. Navigate to 

``` 
cd ~/.ipython/profile_defaults/startup
```

and within there is a `README` file. The README file states that the when you launch a jupyter notebook it will execute any python script in this directory.

So what does that mean? You can have pre-defined functions that exist here and I usually wrap it in a file called `utils.py`

```
touch utils.py
``` 

Here is an example where I place my imports and functions I repeatedly use such as:

```

# Imports
# -------
import os
import MDAnalysis as mda
import pandas as pd
import numpy as np

def find_magnitude(x, y, z):
    
    # Some Code

    return magnitude 
```

Restart your kernel and the functions and imports will be available across notebooks. For an added bonus you can symlink the file to your current directory so you don't always have to navigate to the `.ipython` directory

```
ln -s ~/.ipython/profile_defaults/startup/utils.py path/to/current/directiory/utils.py
```
