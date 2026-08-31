# Code-Aster-MPI-in-Singularity-of-SM2025
31.08.2025 Repository created

____________________________________________________________________________________________________________________
Download container built according to recipe below at: **update link**
____________________________________________________________________________________________________________________



In the following tutorial we will show how to build the MPI Version of Code Aster 17.5 inside the Singularity Container of Salome-Meca 2025. This work is largely based on https://github.com/jcugnoni/SalomeMeca2024_Code-AsterMPI only with some updates and minor changes in the process.

This recipe and the resulting container were tested in Ubuntu 24.04 LTS. Please be aware, that some slight modifications might be necessary when using Ubuntu 24.04 LTS with an Nvidia GPU. One well known caveat are incompatible GLIBC libraries. Errors similar to 

> SALOME_Session_Server_No_Server: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.38' not found (required by /.singularity.d/libs/libGLX.so.0)

appear and Salome-Meca would not launch. The solution is to comment some of these libs in the nvliblist.conf file of your singularity or apptainer installation. Have a look at the code aster forum for this at https://code-aster.org/forum/ . In our setup these libs have to be commented:

>#libEGL.so
>
>#libGLdispatch.so
>
>#libGL.so
>
>#libGLX.so

____________________________________________________________________________________________________________________
Let's start with the build process. For this build we use singularity-ce version 4.4.2-noble .

Download the necessary Singularity Container of Salome-Meca 2025 from https://open-simulation-center.org/fr/downloads/code_aster/SALOME_MECA/2025

and place it in your home folder. The file size is ~5.7GiB. The exact filename is salome_meca-lgpl-2025.1.0-1-20251026-scibian-12.sif .

Open a bash terminal and

```
singularity --version
```

to check your version, it should be around the version above (version 3 should be avoided). If necessary, upgrade singularity.

We need a new dir for this whole process so

```
mkdir SalomeMeca2025_MPI
```

Move the downloaded container into the new dir and cd into dir:

```
mv salome_meca-lgpl-2025.1.0-1-20251026-scibian-12.sif SalomeMeca2025_MPI
cd SalomeMeca2025_MPI
```

Build the sandbox we'll be working in with

```
singularity build --sandbox SalomeMeca2025_mpi salome_meca-lgpl-2025.1.0-1-20251026-scibian-12.sif
```

Enter the sandbox in writable mode

```
singularity shell --writable SalomeMeca2025_mpi
```
You are now in the container, the terminal should show "Singularity>". We need a new dir inside the container for the source code of Coder_Aster:

```
cd /opt/
mkdir codeaster
cd codeaster
```

The current most modern "single digit" version of Coder_Aster is 17.5, so we want to check out this version with git.

```
git clone --branch 17.5.0 https://gitlab.com/codeaster/src.git
git clone https://gitlab.com/codeaster/devtools.git
```

Let'S build this version, all the correct prerequisites are already present in the container. You can check this on Code_Aster's GitLab page: https://gitlab.com/codeaster-opensource-documentation/opensource-installation-development/-/blob/main/devel/changelog.md . 

```
cd src
./waf configure
./waf install -j 8
./waf install test -n zzzz506c
```

The test in the end should show something like " 'test' finished successfully (3.546s) ". 

We need to register CA 17.5 in Salome Meca so it can be chosen in the drop-down menu:

```
echo "vers : stable_mpi:/opt/codeaster/install/mpi/share/aster" >> /opt/salome-meca/2025/V2025.1.0_scibian_univ/tools/Code_aster_frontend-202510/etc/codeaster/aster
```

Fixing of the as_run version is also necessary for the new mpi version:

```
mv /usr/local/bin/as_run /usr/local/bin/as_run_23
ln -s /opt/salome-meca/2025/V2025.1.0_scibian_univ/tools/Code_aster_frontend-202510/bin/as_run /usr/local/bin/as_run
```

Verify the last command with

```
ls -l /usr/local/bin/as_run
```

It should show " /usr/local/bin/as_run -> /opt/salome-meca/2025/V2025.1.0_scibian_univ/tools/Code_aster_frontend-202510/bin/as_run "

Fixing of run_aster_main.py:
Make a copy of run_aster_main.py in case we need to go back, shouldn't be necessary though

```
cd /opt/codeaster/install/mpi/lib/aster/run_aster
cp run_aster_main.py run_aster_main.orig
```

We need to edit this run_aster_main.py, as it is very long and we only have nano inside the container we directly jump to line 480 with:

```
nano run_aster_main.py
```
> CTRL+SHIFT 7

shows " Entrez: numéro de ligne, numéro  de colonne: "

> type '480' and press ENTER

Editor should jump to line 480, search for the line beginning with "proc = ......." and comment this line:
>                # proc = run(cmd, shell=True, check=False)

and/or replace it with this code
>                cmdpfx ="lst=`env | grep OMPI_ | cut -d = -f 1`; for item in $lst; do echo 'unset ' $item; unset $item; done; export PATH=/usr/local/bin:$PATH; "
>                proc = run(cmdpfx+cmd, shell=True, check=False, capture_output=False)

If you are not familiar with Python, the whitespace (spaces before code must remain the same, it is exactly like shown here) is most important. If you don't keep it exactly like it was, this won't work correctly. 

Save the file with CTRL + O and pressing ENTER, exit nano with CTRL+X (CTRL+S and then CTRL+X also works).

Exit the container, the indicator should change back to the normal bash of your user:
```
exit
```

Write the container file with

```
singularity build SalomeMeca2025_mpi.sif SalomeMeca2025_mpi
```

If all went well, move the container back to your user's home

```
mv alomeMeca2025_mpi.sif ~/
cd
```

You can launch this container with (the binds are necessary for Asterstudy to work correctly, as is using the explicit command for salome. Why? WDK):

```
singularity exec --nv \
  --bind /tmp:/tmp \
  --bind /tmp:/local00/tmp \
  SalomeMeca2025_mpi.sif \
  /opt/salome-meca/2025/V2025.1.0_scibian_univ/prerequisites/Cea_archive-9130/salome
```

If you do not have an Nvidia GPU remove the "--n" from above command.

Salome-Meca 2025 should launch correctly. The version drop-down menu in Asterstudy should feature the new version:


<p align="center">
  <img src="https://github.com/user-attachments/assets/b64dc94a-2357-4725-8836-c8bf11012b87" style="width:25%; height:auto;" />
</p>

A blank run of Asterstudy should show "Parallélisme MPI : actif" in the log:

<p align="center">
  <img src="https://github.com/user-attachments/assets/399d963f-35e5-4006-81d1-59af43a97334" style="width:75%; height:auto;" />
</p>

If all this worked correctly, you are ready to use Coder_Aster 17.5 with MPI directly in Salome-Meca 2025.

Have fun,

emefff@gmx.at



