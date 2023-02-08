# hyStrath-Singularity - Rarefied Gas Dynamics Code Based on [OpenFOAM](openfoam.com)

Singularity (Apptainer) of hyStrath, mainly developed by [Dr. Vincent Casseau](https://hystrath.github.io/). Includes
compressible CFD and particle-based DSMC solvers for space related and atmospheric reentry modelling.

Current stable release: **hystrath-1706**

## 1. Installation

### Build Singularity

You need to have Singularity installed. Follow [this guide](install_Singularity.md) for instructions on how to install
Singularity on Linux/Ubuntu.

### Pull Singularity Image

Directly pull it from the official [Singularity library](https://cloud.sylabs.io/library/inflowencer/openfoam/hystrath):

```sh
mkdir -p ~/containers/singularity && cd ~/containers/singularity
singularity pull library://inflowencer/openfoam/hystrath:1706
```

### Setup OpenMPI

For optimal performance and compatibility, the host MPI and image MPI should be the same. hystrath-1706 uses 
**openmpi-4.1.4**.

```sh
mkdir -p ~/openmpi && cd ~/openmpi
wget https://download.open-mpi.org/release/open-mpi/v4.1/openmpi-4.1.4.tar.gz && tar -xzf openmpi-4.1.4.tar.gz
cd openmpi-4.1.4 && ./configure --prefix=$(pwd)
```

## 2. Workflow

### Setup your case inside a folder

```sh
mkdir -p ~/hystrath-runs/Mach_20_cylinder
cd ~/hystrath-runs/Mach_20_cylinder

# Case structure should look like this
Mach_20_cylinder/
├── 0
├── Allpre
├── Allrun
├── constant
└── system
```

### Preprocessing

Usually this includes the **creation or conversion of a mesh** and the **domain decomposition**.
Create an `Allpre` file inside your working directory:

```sh
#!/bin/bash
source /usr/lib/openfoam/OpenFOAM-v1706/etc/bashrc  # Activate OpenFOAM

# Remove old preprocessing log files
rm -f log.*

blockMesh
snappyHexMesh
checkMesh
decomposePar -cellDist -force -latestTime
```

Don't forget to make it executable `chmod +x Allpre`. Now we can execute the image to run our `Allpre` script.

```sh
singularity exec ~/containers/singularity/hystrath_1706.sif ./Allpre
```

Any changes or files written during image run-time will be written to the host.

### Solving

For this, we need an `Allrun` file containing:

```sh
#!/bin/bash
source /usr/lib/openfoam/OpenFOAM-v1706/etc/bashrc  # Activate OpenFOAM
rm -f log.hy2Foam  # Remove old solver log files

hy2Foam -parallel  # Notice that we do not specify mpirun here
```

To run the solver in parallel, we call the image with `mpirun` and specify the Allrun script:

```sh
mpirun -np 8 singularity exec ~/containers/singularity/hystrath_1706.sif ./Allrun
```

If you have different MPI versions on your pc, you can directly call the `openmpi-4.1.4` executable:

```sh
/home/user/openmpi/openmpi-4.1.4/bin/mpirun -np 8 singularity exec ~/containers/singularity/hystrath_1706.sif ./Allrun
```

where `user` must be replaced with your username.

### Postprocessing

In this stage you would postprocess your case and optionally reconstruct the decomposed `processor` directories.
Create an `Allpost` script:

```sh
source /usr/lib/openfoam/OpenFOAM-v1706/etc/bashrc  # Activate OpenFOAM

hy2Foam -postProcess -dict 'system/surfaceCoefficients' -latestTime
reconstructPar
# Optionally, create VTK files of the case 
foamToVTK
```

To postprocess your case, you can then run:

```sh
singularity exec ~/containers/singularity/hystrath_1706.sif ./Allpre
```

## 3. Examples

Examples for this code can be found in the [hyStrath-Examples](https://github.com/inflowencer/hyStrath-Examples) repo.

## 4. Tips and Tricks

It is convenient to specify aliases for certain Singularity commands. Add those lines to your `~/.bashrc` or `~/.zshrc`:

```sh
alias s="singularity"
alias spre="singularity exec ~/containers/singularity/hystrath_1706.sif ./Allpre"
alias spost="singularity exec ~/containers/singularity/hystrath_1706.sif ./Allpost"
function sprun() { 
    /home/user/openmpi/openmpi-4.1.4/bin/mpirun -np $1 singularity exec ~/containers/singularity/hystrath_1706.sif ./Allrun
}
```

If you have your `Allpre` and `Allrun` scripts correctly setup, you can run the case much easier with these aliases.
For instance, to run the [preprocessing tasks](#run-the-preprocessing-tasks), you can just enter `spre`.

To run a case, you would execute `sprun 8` to run with 8 processors. And finally, for postprocessing you can execute
`spost`.