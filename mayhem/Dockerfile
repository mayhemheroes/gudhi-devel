FROM ubuntu:20.04 as builder

RUN apt-get update
# DEBIAN_FRONTEND=noninteractive is used to prevent installation of
# package tzdata to ask for timezone
ENV DEBIAN_FRONTEND=noninteractive

# Required for Gudhi compilation
RUN apt-get install -y make \
    g++ \
    cmake \
    graphviz \
    perl \
    texlive-bibtex-extra \
    biber \
    libboost-all-dev \
    libeigen3-dev \
    libgmp3-dev \
    libmpfr-dev \
    libtbb-dev \
    libcgal-dev \
    locales \
    python3 \
    python3-pip \
    python3-pytest \
    python3-tk \
    python3-pybind11 \
    libfreetype6-dev \
    pkg-config \
    curl

ARG CGAL_VERSION="5.3.1"

RUN curl -LO "https://github.com/CGAL/cgal/releases/download/v${CGAL_VERSION}/CGAL-${CGAL_VERSION}.tar.xz" \
    && tar xf CGAL-${CGAL_VERSION}.tar.xz \
    && mkdir build \
    && cd build \
    && cmake -DCMAKE_BUILD_TYPE=Release ../CGAL-${CGAL_VERSION}/ \
    && make install \
    && cd .. \
    && rm -rf build CGAL-${CGAL_VERSION}

RUN pip3 install \
    numpy \
    matplotlib \
    scipy \
    Cython \
    POT \
    scikit-learn

# apt clean up
RUN apt-get autoremove && rm -rf /var/lib/apt/lists/*

ARG GUDHI_VERSION="3.4.1"

COPY . /repo
WORKDIR /repo/build
RUN cp /repo/mayhem/src_CMakeLists.txt /repo/src/CMakeLists.txt
RUN cp /repo/mayhem/test_CMakeLists.txt /repo/src/Alpha_complex/test/CMakeLists.txt
RUN cmake -DCMAKE_BUILD_TYPE=Release -DWITH_GUDHI_PYTHON=OFF -DPython_ADDITIONAL_VERSIONS=3 ..
RUN make -j8

FROM ubuntu:20.04 as package
RUN apt-get update
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get install -y \
    libboost-all-dev \
    libmpfr-dev \
    libtbb-dev 
COPY --from=builder /repo/build/src/Alpha_complex/utilities/alpha_complex_persistence /
