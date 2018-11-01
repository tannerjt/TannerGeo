---
layout: post
title:  "ArcGIS API for Python on Mac"
author: Joshua Tanner
description: Developer setup for the modern Esri GIS developer.
image: assets/images/python.jpg
---

Mac's currently ship with Python 2.7 as the default install.  The [ArcGIS API for Python](https://developers.arcgis.com/python/guide/system-requirements/) requires at least Python 3.5.  However, I don't want to remove 2.7, but rather have the ability to switch between multiple versions of Python.  

This is the main goal of [pyenv](https://github.com/pyenv/pyenv).  It is much like ruby's environment manager (*rbenv*).  You can change the version of python on a

+ per user basis
+ per project basis
+ with an environment variable
+ globally

Pyenv, however, is not like virtualenv, so we will want to still manage our own virtual environments with [pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv).

## Install

I am using homebrew, a package manager for Mac. ([see docs](https://github.com/pyenv/pyenv#homebrew-on-mac-os-x))

```bash
$ brew update
$ brew install pyenv
```

You can view a full list of Python versions available to install through pyenv [here](https://github.com/pyenv/pyenv/blob/master/COMMANDS.md).

Also, to configure pyenv peroperly, add this to your *.bash_profile* and restart your terminal.

```bash
if which pyenv > /dev/null; then eval "$(pyenv init -)"; fi
```

Let's go ahead and install python 3.5.0.

```bash
$ pyenv install 3.5.0
```

If we wanted to uninstall, we could do:

```bash
$ pyenv uninstall 3.5.0
```

## Set Default Version of Python

### Golbal

Used in all shells

```bash
$ pyenv global 3.5.0
```

### Local

For specific application, writes to file called *.python-version*.  This overrides the global version for current directory and all sub-directories.

```bash
$ pyenv local 3.5.0
```

### Shell

Current shell session

```bash
$ pyenv shell 3.5.0
```

### Anaconda

Anaconda is a package manager, environment manager, and Python distribution.

I'm going to install minconda, which is a slimmed down version of anaconda.  Lets get the latest version.

```bash
$ pyenv install miniconda3-latest
$ pyenv local miniconda3-latest
```

### Work within a virtual environment

To keep things clean and orderly, let's work within a Python virtural environment by installing [pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv)

```bash
$ brew install pyenv-virtualenv
```

and add the following to our *.bash_profile*

```bash
eval "$(pyenv virtualenv-init -)"
```

This should automatically activate out configured python environment when we navigate to the project directory.  This is not required, but really useful.

Now, we can create a virtualenv for a specific python version.

```bash
$ pyenv virtualenv miniconda3-latest my-conda-venv
```

This will create a new virtual environment based on the miniconda3-latest Python install in pyenv.

### Install **arcgis** package

First, lets activate our newly created virtual environement based on miniconda.

```bash
$ pyenv activate my-conda-venv
```

Let's download and install the ArcGIS API in our conda environment.

```bash
$ conda install -c esri arcgis
```

You can test the installation but trying to import the GIS package.  You can run an interactive shell by typing **python** into your termnial.

```python
from arcgis.gis import GIS
```

When we want to deactivate our Python virtual environment, we can simply type:

```bash
$ pyenv deactivate
```

### Jupyter Notebook

Jupyter Notebooks are a popular way to visualize and share your code in a readable informative manner.  Let's also install jupyter with our miniconda environment.

```bash
$ conda install jupyter
```

Now, we can run our Jupyter notebook in the browser.

```bash
$ jupyter notebook
```

Create a new file and test your setup with the following example:

```python
from arcgis.gis import GIS
gis = GIS()
gis.map()
```

### Conclusion

If everything worked properly, you should have now done some really cool things:

+ Used pyenv to manage different installations of Python
+ Managed virtual environments with pyenv-virtualenv
+ Installed miniconda, the lightweight version of anaconda
+ Used the conda package manager to install the Python for ArcGIS library
+ Installed and ran a local Jupyter notebook server
+ Built a small example notebook showcasing the power of Jupyter notebook and ArcGIS

Now, we can confidently play around with new packages and versions of Python without breaking any of the global settings on your mac.

Awesome!
