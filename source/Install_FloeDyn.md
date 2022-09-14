# Install FloeDyn

## Prerequisites

The prerequisites given in the 'readme': 
* python3
* C++ compiler
* gmp
* boost, boost$\_$system, boost$\_$program$\_$options, 
* libboost-all-dev
* eigen
* matio
* hdf5 (c++)
* cgal
* mpfr
* cereal
You also need the following package for mpi version and sequential version :
* sagemath
* libcgal-dev
* libopenmpi-dev
* libopenmpi-bin
* hdf5 for python ( pip install h5py )
With mac os, some special requirement on the necessary version of package are given in the doxygen file (refman.pdf page 2)


## Install FloeDyn

Command to install FloeDyn are recall in the readme file. This section also discusses common problems that may arise during the installation and how to solve it.
* clone git repository from gricad page
```
git clone git@gricad-gitlab.univ-grenoble-alpes.fr:Mo_MIZ/Floe_Cpp.git
```
* go in FLOE\_CPP directory and 
```
python3 ./waf configure --prefix=<where-you-want-to-install>
```
for example, '<where-you-want-to-install>' can be replace by '\$HOME/Install/FloeDyn'
* if all packages are installed and config is a success. Build :
```
python3 ./waf --target FLOE -v
```
    
Two currents problems observed for the installation on ubuntu :
* There may be problems concerning the use of mpi in the code, even during the compilation of the non parallel version. Solve the problem by :
```
export CXX=mpic++
```
Then come back to the configure step
* If an error message appear :  */usr/bin/ld  cannot find  -lhdf5*. The path for lhdf5 is actually in hdf5/serial. Write
```
export LIBRARY_PATH=/usr/lib/x86_64-linux-gnu/hdf5/serial
```
Then come back to the configure step.

**Warning**: Before run any simulations, you have to create a directory *outputs* in the io florder :
```
mkdir io/outputs
```

There exist three version of FloeDyn :
* The classic one, which installation has been explain just above
* FloeDyn with periodic boundary conditions. Then the build command must be replace by :
```
python3 ./waf --target FLOE_PBC -v
```
* The parallel version
```
python3 ./waf --target FLOE_MPI -v
```
