# hyStrath-Singularity - Rarefied Gas Dynamics Code Based on [OpenFOAM](openfoam.com)

Singularity (Apptainer) of hyStrath, developed by [Dr. Vincent Casseau](https://hystrath.github.io/). Includes compressible CFD
and particle-based DSMC solvers for space related and atmospheric reentry modelling.

## 1. Installation

You need to have Singularity installed. Follow [this guide](install_Singularity.md) for instructions on how to install
Singularity on Ubuntu.

### Official Singularity Image (recommended)

You can directly pull it from the official [Singularity library](https://cloud.sylabs.io/library/inflowencer/openfoam/hystrath):

```sh
mkdir -p ~/images/singularity && cd ~/images/singularity
singularity pull library://inflowencer/openfoam/hystrath
```

### From GitHub Repo

Alternatively, you can build it yourself from this GitHub repo. Clone the repo and build using:

```sh
cd ~/ && git clone https://github.com/inflowencer/hyStrath-Docker.git && cd hyStrath-Docker
docker build -t hystrath .
```

## 2. Workflow

1. Setup your case inside a folder where you want to run it

   ```sh
   mkdir -p ~/hystrath-runs/viking-reentry && cd ~/hystrath-runs/viking-reentry
   ```

2. Run the docker container and mount the **current host directory** to the container directory `/run`

   ```sh
   docker run -it --mount src="$(pwd)",target=/run,type=bind hystrath:latest
   ```

   Any changes or files written to the container `/run` folder are going to be written to the host
   `~/hystrath-runs/viking-reentry` folder.

## 3. Run an Example

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
    ```