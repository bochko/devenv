FROM ubuntu:latest

ARG init_dir="/tmp/init"
ARG gcc="https://ftp.gnu.org/gnu/gcc/gcc-11.2.0/gcc-11.2.0.tar.gz"
ARG cmake="https://github.com/Kitware/CMake/releases/download/v3.22.1/cmake-3.22.1.tar.gz"
ARG ninja="https://github.com/ninja-build/ninja/archive/refs/tags/v1.10.2.tar.gz"
ARG ccache="https://github.com/ccache/ccache/releases/download/v4.5.1/ccache-4.5.1.tar.gz"

# update system
RUN apt -y update
RUN apt -y dist-upgrade

# install package dependencies
RUN apt -y install \
libgmp-dev \
libmpfr-dev \
libmpc-dev \
libisl-dev \
libzstd-dev \
gcc \
g++ \
make \
file \
libssl-dev \
libhiredis-dev \
binutils \
libc-dev

RUN mkdir -p "${init_dir}"
ADD ${gcc} "${init_dir}/gcc.tar.gz"
ADD ${cmake} "${init_dir}/cmake.tar.gz"
ADD ${ninja} "${init_dir}/ninja.tar.gz"
ADD ${ccache} "${init_dir}/ccache.tar.gz"

# GCC 11
RUN mkdir -p "${init_dir}/gcc"
RUN tar -xvf "${init_dir}/gcc.tar.gz" -C "${init_dir}/gcc" --strip-components=1
RUN mkdir -p "${init_dir}/gcc/build"
RUN cd "${init_dir}/gcc/build" && \
"${init_dir}/gcc/configure" \
--prefix="/usr/local" \
--disable-multilib \
--enable-shared

RUN cd "${init_dir}/gcc/build" && \
make -j$(($(cat /proc/cpuinfo | grep processor | wc -l)*2))

RUN cd "${init_dir}/gcc/build" && \
make install

# # Env
RUN printf "/usr/local/lib64" >> /etc/ld.so.conf.d/devvm.conf
RUN ldconfig

# CMake
RUN mkdir -p "${init_dir}/cmake"
RUN tar -xvf "${init_dir}/cmake.tar.gz" -C "${init_dir}/cmake" --strip-components=1
RUN cd "${init_dir}/cmake" && \
./bootstrap --parallel=$(($(cat /proc/cpuinfo | grep processor | wc -l)*2)) \
-- \
-DCMAKE_BUILD_TYPE:STRING=Release \
-DCMAKE_INSTALL_PREFIX:STRING=/usr/local
RUN cd "${init_dir}/cmake" && \
make -j$(($(cat /proc/cpuinfo | grep processor | wc -l)*2))
RUN cd "${init_dir}/cmake" && \
make install

# Ninja
RUN mkdir -p "${init_dir}/ninja"
RUN tar -xvf "${init_dir}/ninja.tar.gz" -C "${init_dir}/ninja" --strip-components=1
RUN cd "${init_dir}/ninja" && \
cmake -B build -DCMAKE_BUILD_TYPE:STRING=Release
RUN cd "${init_dir}/ninja" && \
cmake --build build \
--parallel $(($(cat /proc/cpuinfo | grep processor | wc -l)*2))
RUN cd "${init_dir}/ninja" && \
./build/ninja_test
RUN cd "${init_dir}/ninja" && \
cmake --install build --prefix /usr/local

# CCache
RUN mkdir -p "${init_dir}/ccache"
RUN tar -xvf "${init_dir}/ccache.tar.gz" -C "${init_dir}/ccache" --strip-components=1
RUN cd "${init_dir}/ccache" && \
cmake -B build -DCMAKE_BUILD_TYPE:STRING=Release
RUN cd "${init_dir}/ccache" && \
cmake --build build \
--parallel $(($(cat /proc/cpuinfo | grep processor | wc -l)*2))
RUN cd "${init_dir}/ccache" && \
cmake --install build --prefix /usr/local

# Cleanup
RUN apt -y purge gcc g++
RUN rm -rf "${init_dir}"
