FROM mcr.microsoft.com/devcontainers/base:ubuntu-22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV PDK_ROOT=/opt/pdk
ENV VOLARE_HOME=/opt/volare
ENV DISPLAY=:1
ARG SKY130_VER=c6d73a35f524070e85faff4a6a9eef49553ebc2b

# -----------------------------
# System & build dependencies
# -----------------------------
RUN apt-get update && apt-get install -y --no-install-recommends \
    git build-essential autoconf automake libtool m4 \
    tcl tcl-dev tk tk-dev libcairo2-dev \
    libx11-dev libxext-dev libxrender-dev libxpm-dev libxaw7-dev \
    libreadline-dev libncurses-dev flex bison gawk tcsh \
    python3 python3-pip pkg-config curl ca-certificates \
    xfce4 xfce4-terminal x11vnc xvfb novnc websockify supervisor dbus-x11 \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# -----------------------------
# Build and install MAGIC
# -----------------------------
WORKDIR /tmp/build
RUN git clone https://github.com/RTimothyEdwards/magic.git && \
    cd magic && \
    ./configure --with-tcl=/usr --with-tk=/usr --enable-cairo --without-opengl && \
    make -j"$(nproc)" && make install && \
    cd / && rm -rf /tmp/build

# -----------------------------
# Build and install NETGEN
# -----------------------------
WORKDIR /tmp/build
RUN git clone https://github.com/RTimothyEdwards/netgen.git && \
    cd netgen && ./configure && make -j"$(nproc)" && make install && \
    cd / && rm -rf /tmp/build

# -----------------------------
# Build and install NGSPICE
# -----------------------------
WORKDIR /tmp/build
RUN apt-get update && apt-get install -y --no-install-recommends \
    autoconf automake libtool pkg-config bison flex \
    libxmu-dev libfftw3-dev && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /tmp/build
RUN git clone https://git.code.sf.net/p/ngspice/ngspice && \
    cd ngspice && mkdir -p release && cd release && \
    ../autogen.sh && \
    ../configure --with-x --enable-cider --enable-openmp && \
    make -j"$(nproc)" && make install && \
    cd / && rm -rf /tmp/build

# -----------------------------
# Install Sky130 PDK (pinned)
# -----------------------------
RUN python3 -m pip install --no-cache-dir --upgrade volare && \
    mkdir -p "$PDK_ROOT" "$VOLARE_HOME" && \
    volare enable --pdk sky130 ${SKY130_VER} && \
    ln -s "${VOLARE_HOME}/sky130/${SKY130_VER}/pdk/sky130A" "${PDK_ROOT}/sky130A"

ENV MAGIC_TECHPATH="$PDK_ROOT/sky130A/libs.tech/magic"
ENV NETGEN_SETUP="$PDK_ROOT/sky130A/libs.tech/netgen/sky130A_setup.tcl"

# -----------------------------
# noVNC desktop environment
# -----------------------------
COPY supervisord.conf /etc/supervisor/conf.d/vnc.conf

EXPOSE 6080
CMD ["/usr/bin/supervisord", "-n"]
