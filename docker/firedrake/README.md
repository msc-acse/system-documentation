The Dockerfile in this directory creates a docker image with a local firedrake installation and mpltools installed.
It is based on jupyter/scipy-notebook (see https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html)
providing all standard scipy, matplotlib, sympy, etc. packages. 

The firedrake installation is a python venv (as usual) built inside a seperate conda environment (called firedrake). The
python executable from the python venv however is isolated for most purposes (python paths, etc.) so that packages installed
in the conda environment are not available in the firedrake venv. Any additional packages need to be installed using pip
(from the venv). In the terminal the activation scripts are hooked up such that when the conda firedrake environment is activated
(source activate firedrake), the venv is automatically also activated, and deactivating the conda environment (source deactivate)
also deactivates the venv. In jupyter the python from the venv is available as a separate kernel labeled "Python 3 (firedrake)". 
The default "Python 3" provides the base conda environment from jupyter/scipy-notebook which has all the usual plus 
mpltools. mpltools has also been installed in the firedrake venv.
