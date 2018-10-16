# DockerFile for a firedrake + jupyter container

# Use a jupyter notebook base image
FROM jupyter/scipy-notebook

# This DockerFile is looked after by
MAINTAINER Stephan Kramer <s.kramer@imperial.ac.uk>

# Update the system
USER root
# install the necessary:
# curl: to download the firedrake install script
# vim: because no container should be without
# tzdata: not sure
# openssh-client: mpi needs something to login (unless you want fiddle with plm_rsh_agent)
# other packages: copied from firedrake-install script
RUN apt-get update \
    && apt-get -y dist-upgrade \
    && apt-get -y install curl tzdata vim \
                 openssh-client build-essential autoconf automake \
                 cmake gfortran git libblas-dev liblapack-dev \
                 libopenmpi-dev libtool mercurial openmpi-bin \
                 python3-dev python3-pip python3-tk python3-venv \
                 zlib1g-dev libboost-dev \
    && rm -rf /var/lib/apt/lists/*

# the default user in jupyter containers
USER jovyan
RUN pip install mpltools
# we copy the base conda environment, but without hdf5
# the conda hdf5 package does not work with firedrake
# ans1crypto appears to be no longer available from conda-forge
# note: it would seem more natural to start from a minimal conda environment
# with only python, pip and ipython. But after trying out various different combinations
# I gave up on that. This is due to numerous bugs in various versions of the packages, e.g.:
# * broken LDFLAGS and CFLAGS https://github.com/conda-forge/python-feedstock/issues/197
# * ensurepip is missing (it's fine to say pip doesn't work, but after installing the conda pip package, ensurepip really ought to function)
# Firedrake, happens to install correctly however in the jupyter/scipy-notebook conda base environment though (without hdf5),
# so we'll just use that
RUN conda list --explicit | grep -v hdf5 | grep -v h5py | grep -v asn1crypto > spec-file.txt
RUN conda create -n firedrake --file spec-file.txt
RUN rm spec-file.txt
# the firedrake venv is placed inside the conda environment: /opt/conda/envs/firedrake/firedrake-venv
WORKDIR /opt/conda/envs/firedrake/
RUN curl -O https://raw.githubusercontent.com/firedrakeproject/firedrake/master/scripts/firedrake-install
# the --with-zlib is a temp. work-around (should be fixed some time soon in firedrake)
RUN bash -c "source activate firedrake && python3 firedrake-install --no-package-manager --disable-ssh --install pyadjoint --install thetis --venv-name=firedrake-venv"

# automatically activate the firedrake venv when activating the firedrake conda environment
# this only changes behaviour in the (bash) shell
RUN mkdir -p /opt/conda/envs/firedrake/etc/conda/activate.d
# we don't want the firedrake venv to also prepend (firedrake-venv) in addition to (firedrake) from the conda environment:
RUN echo export VIRTUAL_ENV_DISABLE_PROMPT=1 > /opt/conda/envs/firedrake/etc/conda/activate.d/00_virtual_env_disable_prompt.sh
# run the actual firedrake venv activation script when activating the conda firedrake env
RUN cp /opt/conda/envs/firedrake/firedrake-venv/bin/activate /opt/conda/envs/firedrake/etc/conda/activate.d/01_firedrake-venv.sh
RUN mkdir -p /opt/conda/envs/firedrake/etc/conda/deactivate.d
# deactivate is a bash function defined by the firedrake venv activation script
RUN echo deactivate > /opt/conda/envs/firedrake/etc/conda/deactivate.d/01_firedrake-venv.sh
# deactivate removes the firedrake-venv/bin from PATH,
# but then conda overwrites the PATH afterwards with only the /opt/conda/envs/firedrake/bin/ taken out (leaving firedrake-env/bin)
# this is another bug (fixed in later versions, but these have their own issues), that we work around here, 
# by adding the following hack to the conda-env deactivate script
RUN echo "export PATH=$(echo $PATH | sed 's@/opt/conda/envs/firedrake/firedrake-venv/bin:@@')" >> /opt/conda/envs/firedrake/bin/deactivate
# register the firedrake venv python as a kernel for ipython (and thus jupyter)
# for jupyterhub to work, it needs to have jupyterhub installed in that enviroment as well
RUN bash -c "source activate firedrake && pip install jupyterhub ipykernel"
RUN bash -c "source activate firedrake && python -m ipykernel install --user --name firedrake --display-name 'Python 3 (firedrake)'"
# mpltools is only installed (above) in conda, not in the firedrake venv
RUN bash -c "source activate firedrake && pip install mpltools"
WORKDIR /home/jovyan
# avoid warnings about openib not being available in openmpi when running on azure
ENV OMPI_MCA_btl=tcp,self
