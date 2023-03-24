# Copyright (C) 2019 Intel Corporation.  All rights reserved.
# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception

##  Build docker image that consists of gcc, cmake, wasi-sdk & zephyr sdk
FROM ubuntu:22.04
ENV TMP_DIR=/tmp_for_building

RUN mkdir ${TMP_DIR}
COPY resource ${TMP_DIR}
WORKDIR ${TMP_DIR}
##  set compilation environment for wamrc
#    - wamr repo
#    - cmake
#    - wasi-sdk
#    - wamr-sdk

## cmake & make

RUN apt-get update && \
    apt-get install -y git python3 python3-pip wget cmake make ninja-build build-essential --no-install-recommends &&\
    apt-get clean -y &&\
    rm -rf /var/lib/apt/lists/*

##  - download wasi-sdk with wget and set up to /opt/wasi-sdk
RUN wget --progress=dot:giga --no-check-certificate https://github.com/WebAssembly/wasi-sdk/releases/download/wasi-sdk-19/wasi-sdk-19.0-linux.tar.gz && \
    tar -zxvf wasi-sdk-*-linux.tar.gz && \
    mv wasi-sdk-19.0 /opt/wasi-sdk/

## Config git first

RUN git config --global http.sslverify false && \
    git config --global https.sslverify false

##  - clone wamr repo
RUN git clone -b main --depth=1 https://github.com/w1nkNg/wasm-micro-runtime.git

WORKDIR ${TMP_DIR}/wasm-micro-runtime/wamr-compiler
RUN pip3 install requests && \
    ./build_llvm.sh && \
    mkdir build

WORKDIR ${TMP_DIR}/wasm-micro-runtime/wamr-compiler/build
RUN cmake .. &&\
    make &&\
    #  - copy the wamrc to /root
     cp ${TMP_DIR}/wasm-micro-runtime/wamr-compiler/build/wamrc /usr/bin/wamrc && \
     mkdir -p /opt/wamr-sdk && \
     cp -r ${TMP_DIR}/wasm-micro-runtime/wamr-sdk/app /opt/wamr-sdk/ && \
     mv ${TMP_DIR}/wamr_toolchain.cmake /opt/wamr-sdk/app

ENV HOME_DIR=/home/wasm-toolchain

RUN mkdir -p ${HOME_DIR} &&\
    mv ${TMP_DIR}/build_wasm.sh ${HOME_DIR}

# COPY files from BASE image
# COPY --from=BASE /opt/cmake/ /opt/cmake/
# COPY --from=BASE /opt/wamr-sdk/app/ /opt/wamr-sdk/app/
# COPY --from=BASE /opt/wasi-sdk /opt/wasi-sdk/
# COPY --from=BASE /root/wamrc ${HOME_DIR}
# COPY --from=BASE /root/build_wasm.sh ${HOME_DIR}

# RUN ln -s ${HOME_DIR}/wamrc /usr/bin/wamrc
RUN rm -fr ${TMP_DIR}

WORKDIR ${HOME_DIR}
