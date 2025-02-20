# SmartSim OpenMM

This repo contains a SmartSim version of the DeepDriveMD workflow. 

## Installation

We suggest to create a fresh conda environment

```bash
conda create --name openmm python=3.7
```

and then to install packages in the following way:

```bash
pip install cmake tensorflow==2.4.2 numpy==1.19.5 cython smartsim sklearn MDAnalysis parmed tables
```
then download SmartRedis from the CrayLabs repo and build it

``` bash
git clone https://github.com/CrayLabs/SmartRedis.git smartredis
cd smartredis
make lib
pip install .
```
then install git lfs and Swig

```bash
conda install git-lfs swig
```

and SmartSim

``` bash
pip install smartsim
```
Follow the SmartSim docs instructions on how to build the ML backends for GPU.

Finally, given the long list of dependencies, we suggest building OpenMM from source,
using the correct GPU libraries and drivers.


## System-dependent settings and driver scripts

- The code contained in `smartsim_md.py` is written to be run on a Cray XC-50 system running Slurm as a workload manager.
- The code contained in `smartsim_md_thetagpu` is written to be run on Theta GPU, with Cobalt as a workload manager.
Each MD simulation uses one GPU. Since each Theta GPU node has 8 GPUs, 8 MD simulations are launched on the same node,
each one using one GPU and 16 CPUs (rank files are generated to bind tasks to CPUs). The same holds for CVAE training,
where each CVAE is trained on a separate GPU.

Launcher and system constraints (like the flag used to access GPUs), have to be adapted for other systems.

## Running the pipeline

From the repo root directory, you should get an interactive allocation with
a total of (#MD Simulation + #ML Workers + #DB nodes + 1) nodes

All the node counts can be modified in the driver scripts.
From the interactive allocation, you can then run:

```bash
python smartsim_md.py
```

or, on Theta GPU

```bash
python smartsim_md_thetagpu.py
```

Instead of inside an interactive allocation, the drivers can be run inside a batch script, as long as it requests the correct
number of nodes and resources. On a Cray XC-50, a small instance of `summit_md.py` (2 MD nodes, 2 ML nodes, 1 DB node) could be run, for example through the following batch script:

```bash
#SBATCH -N 6
#SBATCH -C P100
#SBATCH --time 02:00:00

module load <your modules>
conda activate <your env>

python smartsim_md.py
```

## Implementation details

The binary file storage in the database is an actively developed feature, and thus it has not yet been thoroughly tested or optimized. It is included in this project as a proof of concept and should not be considered ready for production. For this reason, the user can set the variable BINARY_FILES to 1 in the driver scripts, to write all binary files to disk instead of storing them on the DB.
