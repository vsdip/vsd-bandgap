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
RUN wget http://opencircuitdesign.com/magic/archive/magic-8.3.32.tgz && \
    tar xvfz magic-8.3.32.tgz && \
    cd magic-8.3.32 && \
    ./configure && \
    make && \
    make install && \
    cd / && rm -rf /tmp/build

# -----------------------------
# Install NGSPICE
# -----------------------------
RUN apt-get update && apt-get install -y --no-install-recommends \
    ngspice \
 && rm -rf /var/lib/apt/lists/*

# -----------------------------
# Build and install NETGEN
# -----------------------------
WORKDIR /tmp/build
RUN git clone git://opencircuitdesign.com/netgen && \
    cd netgen && \
    ./configure && \
    make && \
    make install && \
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
