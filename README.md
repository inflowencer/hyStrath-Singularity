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
mkdir -p ~/images/singularity && cd ~/images/singularity
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

### Run the preprocessing tasks

Usually this includes the **creation or conversion of a mesh** and the **domain decomposition**.
Create an `Allpre` file inside your working directory

```sh
#!/bin/bash
source /usr/lib/openfoam/OpenFOAM-v1706/etc/bashrc  # Activate OpenFOAM

# Remove old preprocessing log files
rm -f log.blockMesh
rm -f log.snappyHexMesh
rm -f log.checkMesh
rm -f log.decomposePar

runApplication blockMesh
runApplication snappyHexMesh
runApplication checkMesh
runApplication decomposePar -cellDist -force -latestTime
```

Don't forget to make it executable `chmod +x Allpre`. Now we can execute the image to run our `Allpre` script.

```sh
$ singularity exec ~/images/singularity/hystrath_1706.sif ./Allpre
```

Any changes or files written during image run-time will be written to the host.

### Run the solver

For this, we need an `Allrun` file.

```sh
#!/bin/bash
source /usr/lib/openfoam/OpenFOAM-v1706/etc/bashrc  # Activate OpenFOAM

# Remove old solver log files
rm -f log.hy2Foam

# Notice that we do not specify mpirun here
hy2Foam -parallel
```

To run the solver in parallel, we call the image with `mpirun` and specify the Allrun script:

```sh
$ mpirun -np 8 singularity exec ~/images/singularity/hystrath_1706.sif ./Allrun > log.2>&1
```



<!-- ## 3. Run an Example

### Viking Mars Reentry

1.  Clone the [hyStrath-Examples Collection](https://github.com/inflowencer/hyStrath-Examples) and copy the Viking Mars
    Reentry case to your `hystrath-run` directory on the host

    ```sh
    git clone https://github.com/inflowencer/hyStrath-Examples.git ~/hyStrath-Examples
    cp -r ~/hyStrath-Examples/examples/viking-mars-reentry ~/hystrath-runs/.
    # Switch into the directory 
    cd ~/hystrath-runs/viking-mars-reentry
    ```

2.  Run the docker container and mount the working directory to the container `/run` directory

    ```sh
    docker run -it --mount src="$(pwd)",target=/run,type=bind hystrath:latest
    cd /run
    ```

3.  Run the case

    ```sh
    ./Allrun
    ```

4.  After the case ran through, you can close the docker container and postprocess it

    ```sh
    exit
    paraview &
    # Open the pv.foam file and use `decomposedCase` to postprocess
    ``` -->