## MaTEx Docker: Machine Learning Toolkit for Extreme Scale in Docker
==================================================================

MaTEx is a collection of high performance parallel machine learning and
data mining (MLDM) algorithms using [OpenMPI](https://www.open-mpi.org/) and [TensorFlow](https://www.tensorflow.org/).

Docker provides operating system-level virtualization via containerization.
For more information see [Docker](https://www.docker.com/) and [Wikipedia](https://en.wikipedia.org/wiki/Docker_(software))

The matex-docker project supports MaTEx executing in Docker, thus providing a single set of configurations
and dependencies to build, ship, and run MaTEx on laptops, data center VMs, or the cloud.

### Project Structure
==================================================================

Here is a quick breakdown of how matex-docker is structured. All Dockerfiles are configured
to support both single and multi-container MPI execution. Default is single container.

# benchmarks

MPI4PY benchmark scripts

# dockerfiles

Dockerfile support for CentOS and Ubuntu

# compose

Support for multi-container OpenMPI using SSH and [Docker Compose](https://docs.docker.com/compose/)

# openmpi

User-specific OpenMPI configuration parameter files

# ssh

User-specific SSH configuration files

## Docker Build
==================================================================

MaTEx Docker must be build from the root of the repository

# Example
docker build -f dockerfiles/ubuntu/16.x/Dockerfile .

## Single Container MaTEx Execution
==================================================================

**Clone matex-docker project**

* cd LOCAL_DIR
* git pull MATEX_DOCKER_REPO_URL

**Build and Run Docker container**

* docker build .
* Once docker build is complete
  * docker images
* Take note of the newest Image ID
* Run the Docker container
  * docker run -i -t IMAGE_ID /bin/bash

**Execute MaTEx inside container**

* cd matex/src/deeplearning/tensorflow/cpu/py3.x/
* source activate_mtx.sh
* **VERIFY**
  * From current directory, execute Python manually
    * python
    * import tensorflow as tf
  * Expected Output
    * If no errors or failures, then quit()
* Execute MaTEx Example code (MNIST)
  * cd /matex/src/deeplearning/tensorflow/examples/glibc_after_2.19/MNIST_KERAS/
  * mpirun --allow-run-as-root -mca btl_vader_single_copy_mechanism none -np 4 python keras_lenet3.py

## Multi-Container MaTEx Execution (IN PROGRESS)
==================================================================

Container Cluster orchestration uses `docker-compose`

While containers can in principle be started manually via `docker run`, we suggest that you use
[Docker Compose](https://docs.docker.com/compose/), a command-line tool
to define and run multi-container applications.

We provide a sample `docker-compose.yml` file in the repository:

```
mpi_head:
  image: openmpi
  ports:
   - "22"
  links:
   - mpi_node

mpi_node:
  image: openmpi

```
(Note: the above is docker-compose API version 1)

The file defines an `mpi_head` and an `mpi_node`. Both containers run the same `openmpi` image.
The only difference is, that the `mpi_head` container exposes its SSH server to
the host system, so you can log into it to start your MPI applications.

# Usage

The following command, run from the repository's directory, will start one `mpi_head` container and three `mpi_node` containers:

```
$> docker-compose scale mpi_head=1 mpi_node=3
```
Once all containers are running, you can login into the `mpi_head` node and start MPI jobs with `mpirun`. Alternatively, you can execute a one-shot command on that container with the `docker-compose exec` syntax, as follows:

    docker-compose exec --user mpirun --privileged mpi_head mpirun -n 2 python /home/mpirun/mpi4py_benchmarks/all_tests.py
    ----------------------------------------- ----------- --------------------------------------------------
    1.                                        2.          3.

Breaking the above command down:

1. Execute command on node `mpi-head`
2. run on 2 MPI ranks
3. Command to run (NB: the Python script needs to import MPI bindings)

# Testing

You can spin up a docker-compose cluster, run a battery of MPI4py tests and remove the cluster using a recipe provided in the included Makefile (handy for development):

    make main

## Credits
==================================================================

OpenMPI SSH work based on [docker.openmpi](https://github.com/oweidner/docker.openmpi) and [dispel4py](https://github.com/dispel4py/) by O. Weidner and R. Filgueira