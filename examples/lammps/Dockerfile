# ch-test-scope: full
FROM openmpi
WORKDIR /usr/local/src

# Packages for building.
RUN dnf install -y --setopt=install_weak_deps=false \
    cmake \
    patch \
    python3-devel \
    python3-pip \
    python3-setuptools

# Building mpi4py from source to ensure it is built against our MPI build
# Building numpy from source to work around issues seen on Aarch64 systems
RUN pip3 install --no-binary :all: cython==0.29.14 mpi4py==3.0.3 numpy==1.18.1
#RUN ln -s /usr/bin/python3 /usr/bin/python
# Build LAMMPS.
ENV LAMMPS_VERSION 19Sep19
RUN wget -nv https://lammps.sandia.gov/tars/lammps-$LAMMPS_VERSION.tar.gz
RUN tar xf lammps-$LAMMPS_VERSION.tar.gz
RUN mkdir lammps-${LAMMPS_VERSION}.build
RUN    cd lammps-${LAMMPS_VERSION}.build \
    && cmake -DCMAKE_INSTALL_PREFIX=/usr/local \
             -DCMAKE_BUILD_TYPE=Release \
             -DBUILD_MPI=yes \
             -DBUILD_LIB=on \
             -DBUILD_SHARED_LIBS=on \
             -DPKG_DIPOLE=yes \
             -DPKG_KSPACE=yes \
             -DPKG_POEMS=yes \
             -DPKG_PYTHON=yes \
             -DPKG_USER-REAXC=yes \
             -DPKG_USER-MEAMC=yes \
             -DLAMMPS_MACHINE=mpi \
          ../lammps-${LAMMPS_VERSION}/cmake \
    && make -j $(getconf _NPROCESSORS_ONLN) install
RUN ldconfig
RUN ln -s /usr/local/src/lammps-${LAMMPS_VERSION}/ /lammps
RUN rm -f lammps-$LAMMPS_VERSION.tar.gz

# Patch in.melt to increase problem dimensions.
COPY melt.patch /lammps/examples/melt
RUN patch -p1 -d / < /lammps/examples/melt/melt.patch
# Patch simple.py to uncomment mpi4py calls and disable file output.
# Patch in.simple to increase problem dimensions.
COPY simple.patch /lammps/python/examples
RUN patch -p1 -d / < /lammps/python/examples/simple.patch
