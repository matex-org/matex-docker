# ------------------------------------------------------------
# Use Ubuntu base image
# ------------------------------------------------------------

FROM ubuntu:16.04

# ------------------------------------------------------------
# Install OS dependencies
# ------------------------------------------------------------

RUN apt-get update && yes|apt-get upgrade

RUN apt-get update -y && \
    apt-get install -y --no-install-recommends sudo && \
    apt-get install -y --no-install-recommends wget bzip2 apt-utils \
        openssh-server python-dev python-numpy python-pip python-virtualenv \
        python-scipy build-essential m4 git vim gcc gfortran libopenmpi-dev \ 
        openmpi-bin openmpi-common openmpi-doc binutils && \
    apt-get clean && apt-get purge && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*


# ------------------------------------------------------------
# Install Miniconda
# ------------------------------------------------------------

RUN wget https://repo.continuum.io/miniconda/Miniconda3-4.5.4-Linux-x86_64.sh

RUN bash Miniconda3-4.5.4-Linux-x86_64.sh -b
RUN rm Miniconda3-4.5.4-Linux-x86_64.sh

# Set path to conda
ENV PATH /root/miniconda3/bin:$PATH

# Updating Miniconda packages
RUN conda update conda
RUN conda update --all
RUN conda install python=3.5.2 -y

RUN conda install pip
RUN conda upgrade pip

USER ${USER}

RUN pip install --user -U setuptools


# ------------------------------------------------------------
# Install OpenMPI
# ------------------------------------------------------------

# Add an 'mpirun' user
ENV USER mpirun

RUN adduser --disabled-password --gecos "" ${USER} && \
    echo "${USER} ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

ENV DEBIAN_FRONTEND=noninteractive \
    HOME=/home/${USER}

# Run following commands as root
USER root

RUN wget https://download.open-mpi.org/release/open-mpi/v3.0/openmpi-3.0.0.tar.gz
RUN tar -zxf openmpi-3.0.0.tar.gz
RUN mkdir -p openmpi-3.0.0/build /openmpi
RUN cd /openmpi-3.0.0/build && ../configure --prefix=/openmpi --enable-mpi-cxx --enable-shared CC=`which gcc` CXX=`which g++` && make -j4 && make install
RUN rm -rf /openmpi-3.0.0*

ENV MPI_HOME /openmpi
ENV PATH /openmpi/bin:${PATH}
ENV LD_LIBRARY_PATH /openmpi/lib:${LD_LIBRARY_PATH}

# RUN rm -fr ${HOME}/.openmpi && mkdir -p ${HOME}/.openmpi
# ADD default-mca-params.conf ${HOME}/.openmpi/mca-params.conf
# RUN chown -R ${USER}:${USER} ${HOME}/.openmpi


# ----------------------
# Install MaTEx
# ----------------------

RUN git clone https://github.com/matex-org/matex.git

ENV ANACONDA_HOME=/root/miniconda3
RUN rm -rf /matex/src/deeplearning/tensorflow/cpu/py3.x/matex_tf
RUN echo $MPI_HOME
RUN echo $ANACONDA_HOME
RUN cd /matex/src/deeplearning/tensorflow/cpu/py3.x && chmod a+x /matex/src/deeplearning/tensorflow/cpu/py3.x/install_tf_matex_0.7.0.sh && /matex/src/deeplearning/tensorflow/cpu/py3.x/install_tf_matex_0.7.0.sh


# ------------------------------------------------------------
# Configure SSH
# ------------------------------------------------------------

RUN mkdir /var/run/sshd
RUN echo 'root:${USER}' | chpasswd
RUN sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config

# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile

# Add SSH keys
ENV SSHDIR ${HOME}/.ssh/

RUN mkdir -p ${SSHDIR}

ADD ssh/config ${SSHDIR}/config
ADD ssh/id_rsa.mpi ${SSHDIR}/id_rsa
ADD ssh/id_rsa.mpi.pub ${SSHDIR}/id_rsa.pub
ADD ssh/id_rsa.mpi.pub ${SSHDIR}/authorized_keys

RUN chmod -R 600 ${SSHDIR}* && \
    chown -R ${USER}:${USER} ${SSHDIR}

# ------------------------------------------------------------
# Add MPI4PY Benchmarks
# ------------------------------------------------------------

ENV TRIGGER 1

ADD benchmarks ${HOME}/benchmarks
RUN chown -R ${USER}:${USER} ${HOME}/benchmarks

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
