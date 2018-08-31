# Installing the Anaconda python package

In general, instructions & links are available from https://www.anaconda.com/download. For specific operating systems:

## Windows

Graphical installer only. Option to install Visual Studio Code (**NOT** Visual Studio).

## Mac

Options for graphical or command line installers (see linux instructions for the method with the latter).  Option to install Visual Studio Code (**NOT** Visual Studio).

## Linux

The download link, `https://repo.anaconda.com/archive/Anaconda3-5.2.0-Linux-x86_64.sh`, provides a shell script. This can be installed (in non-interactive mode) by running

    bash Anaconda3-5.2.0-Linux-x86_64.sh -b -f -p PREFIX

where `-b` selects batch mode, `-f` suppresses errors if the install path already exists and `-p PREFIX` specifies the path to install to (defaults to the value of `$PREFIX`, or else `/home/$USER/anaconda3`.  Option to install Visual Studio Code (**NOT** Visual Studio).
