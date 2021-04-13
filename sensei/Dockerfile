FROM srizzi/paraview-ospray

RUN  mkdir /sensei
RUN  mkdir /sensei/build
RUN  mkdir /sensei/src
RUN  mkdir /sensei/install
# need to install some development tools to
# build our code
#RUN  zypper update -y
RUN yum install -y \
    curl \
    gcc gcc-c++ gcc-gfortran \
    wget \
    git \
    sudo \
    vim \
    automake \
    openssl-devel \
    python3-devel \
    openssh-clients \
    libxml2-devel
RUN yum autoremove -y
#RUN  zypper install -y gcc \
#       gcc-c++ \
#       gcc-fortran \
#       ncurses-devel \
#       wget \
#       libXt-devel \
#       freeglut-devel \
#       automake \
#       bind-utils \
#       which \
#       cmake \
#       tar \
#       gzip \
#       git

# MPICH
# install MPICH
ARG MPICH_VERSION=3.2.1

WORKDIR /sensei/src
RUN  wget -q http://www.mpich.org/static/downloads/$MPICH_VERSION/mpich-$MPICH_VERSION.tar.gz \
       && tar xzf mpich-$MPICH_VERSION.tar.gz \
       && rm mpich-$MPICH_VERSION.tar.gz \
       && cd mpich-$MPICH_VERSION \
       && ./configure --prefix=/usr/local/mpich/install --disable-wrapper-rpath \
       && make -j8 \
       && make install
# disable the addition of the RPATH to compiled executables
# this allows us to override the MPI libraries to use those
# found via LD_LIBRARY_PATH

ARG PATH_ORIG=$PATH
ARG LD_LIBRARY_PATH_ORIG=$LD_LIBRARY_PATH

ENV PATH=/usr/local/mpich/install/bin:$PATH
ENV LD_LIBRARY_PATH=/usr/local/mpich/install/lib:$LD_LIBRARY_PATH

ENV PATH=/pvbuild/build/paraview/install/bin:$PATH
ENV LD_LIBRARY_PATH=/pvbuild/build/paraview/install/lib/python3.8:$LD_LIBRARY_PATH

workdir /pvbuild/install
run wget https://github.com/Kitware/CMake/releases/download/v3.18.4/cmake-3.18.4-Linux-x86_64.tar.gz \
 && tar xvzf cmake-3.18.4-Linux-x86_64.tar.gz \
 && rm cmake-3.18.4-Linux-x86_64.tar.gz \
 && mv cmake-3.18.4-Linux-x86_64 cmake

# add cmake binaries to the path
env PATH="/pvbuild/install/cmake/bin:$PATH"

# VTK
#WORKDIR /sensei/src
#RUN git clone --recursive https://gitlab.kitware.com/vtk/vtk.git \
#      && cd vtk \
#      && git checkout v9.0.1 \
#      && git submodule update
#WORKDIR /sensei/build
#RUN mkdir vtk
#WORKDIR vtk
#RUN cmake \
#    -DCMAKE_INSTALL_PREFIX=/sensei/install/vtk \
#    -DCMAKE_BUILD_TYPE=Release \
#    /sensei/src/vtk \
#  && make -j8 \
#  && make install

# SENSEI
WORKDIR /sensei/src
RUN git clone https://gitlab.kitware.com/sensei/sensei.git \
  && cd sensei \ 
  && git checkout develop \ 
  && cd /sensei/build \ 
  && mkdir sensei \ 
  && cd sensei \ 
  && cmake \
     -DENABLE_SENSEI=ON \
     -DCMAKE_INSTALL_PREFIX=/sensei/install/sensei \
     -DENABLE_CATALYST=ON \
     -DParaView_DIR=/paraview \
     -Drkcommon_DIR=/pvbuild/build/paraview/install/lib/cmake/rkcommon-1.5.1 \
     -Dopenvkl_DIR=/pvbuild/build/paraview/install/lib/cmake/openvkl-0.11.0 \
     -DCMAKE_BUILD_TYPE=Release \
    /sensei/src/sensei \
 && make -j8 \
 && make install

# Sensei config
WORKDIR /sensei/src/sensei/tools/sensei_config
RUN sh make_install /sensei/install/sensei

# Setup ENV variables
ENV PATH=$PATH_ORIG:/sensei/install/sensei/bin
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH_ORIG:/sensei/install/vtk/lib64
