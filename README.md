# Code-Aster-MPI-in-Singularity-of-SM2025
31.08.2025 Repository created

____________________________________________________________________________________________________________________
Download container built according to recipe below at: **update link**
____________________________________________________________________________________________________________________



In the following tutorial we will show how to build the MPI Version of Code Aster 17.5 inside the Singularity Container of Salome-Meca 2025. This work is largely based on https://github.com/jcugnoni/SalomeMeca2024_Code-AsterMPI only with some updates and minor changes in the process.

This recipe and the resulting container were tested in Ubuntu 24.04 LTS. Please be aware, that some slight modifications might be necessary when using Ubuntu 24.04 LTS with an Nvidia GPU. One well known caveat are incompatible GLIBC libraries. Errors similar to 

> SALOME_Session_Server_No_Server: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.38' not found (required by /.singularity.d/libs/libGLX.so.0)

appear and Salome-Meca would not launch. The solution is to comment some of these libs in the nvliblist.conf file of your singularity or apptainer installation. Have a look at the code aster forum for this at https://code-aster.org/forum/ . 

