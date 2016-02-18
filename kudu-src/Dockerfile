FROM ubuntu:14.04

####################
# Dependencies
####################

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y  --no-install-recommends \
    git \
    rsync \
    ca-certificates \
    curl \
    patch \
    automake \
    libtool \
    gettext \
    pkg-config \
    g++ \
    make \
    unzip \
    libsasl2-dev \
    libboost-thread-dev \
    python2.7 \
    liboauth-dev && \
    ln -s /usr/bin/python2.7 /usr/bin/python


####################
# Building Kudu
####################

ENV KUDU_VERSION=master \
    KUDU_SRC=/usr/local/src/kudu \
    KUDU_INSTALL_PATH=/opt/kudu


# Note: git and build-essential are required to build Kudu from source
RUN git clone -b ${KUDU_VERSION} --single-branch --depth=1 https://github.com/cloudera/kudu.git ${KUDU_SRC} && \
    cd $KUDU_SRC && \
    export PATH=${KUDU_SRC}/thirdparty/installed/bin:${PATH} && \
    ./thirdparty/build-if-necessary.sh && \
    echo "${KUDU_SRC}/thirdparty/installed/lib" > /etc/ld.so.conf.d/kudu.installed.dev.conf && \
    echo "${KUDU_SRC}/thirdparty/installed-deps/lib" > /etc/ld.so.conf.d/kudu.installed-deps.dev.conf && \
    ldconfig


RUN export PATH=${KUDU_SRC}/thirdparty/installed/bin:${PATH} && \
    cd $KUDU_SRC && \
    mkdir -p build/release && cd build/release && \
    cmake -DNO_TESTS=1 -DKUDU_LINK=dynamic  -DCMAKE_BUILD_TYPE=release ../.. && \
    make -j8 && \
    rm /etc/ld.so.conf.d/kudu.installed.dev.conf  /etc/ld.so.conf.d/kudu.installed-deps.dev.conf && \
    mv ${KUDU_SRC}/build/release ${KUDU_INSTALL_PATH} && \
    echo "${KUDU_INSTALL_PATH}/lib" > /etc/ld.so.conf.d/kudu.dev.conf && \
    mkdir -p ${KUDU_INSTALL_PATH}/thirdparty/installed-deps && \
    cp ${KUDU_SRC}/thirdparty/installed-deps/lib --reflink=auto ${KUDU_INSTALL_PATH}/thirdparty/installed-deps/lib -R && \
    echo "${KUDU_INSTALL_PATH}/thirdparty/installed-deps/lib" > /etc/ld.so.conf.d/kudu.installed-deps.conf && \
    mkdir ${KUDU_INSTALL_PATH}/thirdparty/installed && \
    cp ${KUDU_SRC}/thirdparty/installed/lib --reflink=auto ${KUDU_INSTALL_PATH}/thirdparty/installed/lib -R && \
    echo "${KUDU_INSTALL_PATH}/thirdparty/installed/lib". > /etc/ld.so.conf.d/kudu.installed.conf && \
    ldconfig -v


ENV PATH=${KUDU_INSTALL_PATH}/bin:${KUDU_INSTALL_PATH}/thirdparty/installed-deps/bin:${KUDU_INSTALL_PATH}/thirdparty/installed/bin:${PATH}


VOLUME /data/kudu-master /data/kudu-tserver

####################
# PORTS
####################
#
# https://github.com/cloudera/kudu/blob/master/docs/installation.adoc
#
# TabletServer:
#   7050 = TabletServer RPC Port
#   8050 = TabletServer Web UI
# Kudu Master:
#   7051 = Master RPC Port
#   8051 = Master Web UI
#

ENV KUDU_OPTS=""

COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
EXPOSE 7050 7051 8050 8051
CMD ["help"]
