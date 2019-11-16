Bootstrap: docker
From: ubuntu:latest

%files
    vecdot.c /opt

%environment
    #export OMPI_DIR=/usr/tce/packages/openmpi/openmpi-4.0.0-intel-18.0.1/
    export OMPI_DIR=/opt/ompi
    export SINGULARITY_OMPI_DIR=$OMPI_DIR
    export SINGULARITYENV_APPEND_PATH=$OMPI_DIR/bin
    export SINGULAIRTYENV_APPEND_LD_LIBRARY_PATH=$OMPI_DIR/lib

%post
    echo "Installing required packages..."
    apt-get update && apt-get install -y wget git bash gcc gfortran g++ make file vim python unzip autoconf libtool flex


    # Insalling LibEvent
    mkdir -p /tmp/libevent/
    cd /tmp/libevent/ && wget https://github.com/libevent/libevent/releases/download/release-2.1.11-stable/libevent-2.1.11-stable.tar.gz
    tar -xvf libevent-2.1.11-stable.tar.gz && cd libevent-2.1.11-stable && ./configure --prefix=/usr/ && make && make install

    # Installing PMIx
    mkdir -p /tmp/pmix/
    cd /tmp/pmix/ && wget https://github.com/openpmix/openpmix/archive/v2.2.3.zip && unzip -o v2.2.3.zip 
    cd openpmix-2.2.3/ && ./autogen.pl
    mkdir -p build && cd build # -p is necessary to force overwrite if dir exists
    ../configure --prefix=/usr/ && make install

    ## Install SLURM with PMI support
    mkdir -p /tmp/slurm
    cd /tmp/slurm && wget https://github.com/SchedMD/slurm/archive/slurm-18-08-8-1.tar.gz && tar -xvf slurm-18-08-8-1.tar.gz
    cd /tmp/slurm/slurm-slurm-18-08-8-1 && ./configure --prefix=/usr/ --with-pmix=/usr/ && make install

    echo "Installing Open MPI"
    export OMPI_DIR=/opt/ompi
    export OMPI_VERSION=3.0.1 # 4.0.0 was giving problems with the PMIX version, also 4.0.1
    export OMPI_URL="https://github.com/open-mpi/ompi/archive/v$OMPI_VERSION.tar.gz"
    mkdir -p /tmp/ompi
    mkdir -p /opt
    # Download
    cd /tmp/ompi && wget -O v$OMPI_VERSION.tar.gz $OMPI_URL && tar -xvf v$OMPI_VERSION.tar.gz
    # Compile and install
    cd /tmp/ompi/ompi-3.0.1 && ./autogen.pl && ./configure --prefix=$OMPI_DIR --with-pmix=/usr/ --with-libevent=/usr/ && make install
    # Set env variables so we can compile our application
    export PATH=$OMPI_DIR/bin:$PATH
    export LD_LIBRARY_PATH=$OMPI_DIR/lib:$LD_LIBRARY_PATH
    export MANPATH=$OMPI_DIR/share/man:$MANPATH

    echo "Compiling the MPI application..."
    cd /opt && mpicc -o vecdot vecdot.c -lm
