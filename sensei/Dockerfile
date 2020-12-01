FROM centos:7

RUN  mkdir /sensei
RUN  mkdir /sensei/build
RUN  mkdir /sensei/src
RUN  mkdir /sensei/install
# need to install some development tools to
# build our code
RUN  yum update -y
RUN  yum groupinstall -y "Development Tools"
RUN  yum install -y gcc \
       g++ \
       ncurses-devel \
       wget \
       xorg-x11-server-Xorg xorg-x11-xauth xorg-x11-apps \
       libXt-devel \
       freeglut-devel \
       automake \
       bind-utils \
       which


# CMake 3.12.3
 
RUN  cd /sensei/src
RUN wget https://cmake.org/files/v3.12/cmake-3.12.3.tar.gz \
      && tar xzf cmake-3.12.3.tar.gz \
      && rm cmake-3.12.3.tar.gz \
      && cd cmake-3.12.3 \
      && ./bootstrap \
      && make -j8 \
      && make install


# MPICH
# install MPICH
ARG MPICH_VERSION=3.3

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

# VTK
WORKDIR /sensei/src
RUN wget https://www.vtk.org/files/release/8.2/VTK-8.2.0.tar.gz \
  && tar xzf VTK-8.2.0.tar.gz \
  && rm VTK-8.2.0.tar.gz
WORKDIR /sensei/build
RUN mkdir vtk
WORKDIR vtk
RUN cmake \
    -DCMAKE_INSTALL_PREFIX=/sensei/install/vtk \
    /sensei/src/VTK-8.2.0 \
  && make -j8 \
  && make install

# SENSEI
WORKDIR /sensei/src
RUN git clone https://gitlab.kitware.com/sensei/sensei.git \
  && cd sensei \ 
  && git checkout v2.1.1 \ 
  && cd /sensei/build \ 
  && mkdir sensei \ 
  && cd sensei \ 
  && cmake \
     -DENABLE_SENSEI=ON \
     -DENABLE_PARALLEL3D=OFF \
     -DENABLE_VTK_IO=ON \
     -DENABLE_VTK_MPI=OFF \
     -DCMAKE_INSTALL_PREFIX=/sensei/install/sensei \
     -DVTK_DIR=/sensei/install/vtk/lib64/cmake/vtk-8.2 \
   /sensei/src/sensei \
   && make -j8 \
   && make install

# Sensei config
#ARG CC=cc
#ARG CXX=c++
WORKDIR /sensei/src/sensei/tools/sensei_config
RUN sh make_install /sensei/install/sensei

# Setup ENV variables
ENV PATH=$PATH_ORIG:/sensei/install/sensei/bin
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH_ORIG:/sensei/install/vtk/lib64

