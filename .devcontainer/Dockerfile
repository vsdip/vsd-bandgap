FROM mcr.microsoft.com/devcontainers/base:ubuntu-22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV PDK_ROOT=/opt/pdk
ENV VOLARE_HOME=/opt/volare
ENV DISPLAY=:1
ENV GEOMETRY=1280x800
ENV DEPTH=24
# Pin a known-good sky130 volare version (you validated this one)
ARG SKY130_VER=c6d73a35f524070e85faff4a6a9eef49553ebc2b
# Magic version >= 8.3.105
ARG MAGIC_VER=8.3.311

# --- OS deps, desktop stack, EDA tools from apt ---
RUN apt-get update && apt-get install -y \
    git curl ca-certificates wget build-essential automake autoconf m4 \
    tcl tcl-dev tk tk-dev libcairo2-dev libx11-dev libxrender-dev libxext-dev \
    libxpm-dev libxaw7-dev libreadline-dev libncurses-dev \
    pkg-config flex bison \
    xfce4 xfce4-terminal x11vnc xvfb novnc websockify supervisor dbus-x11 \
    ngspice netgen \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# --- Build & install MAGIC from source (>= 8.3.105) ---
WORKDIR /tmp/build
RUN git clone https://github.com/RTimothyEdwards/magic.git && \
    cd magic && git checkout ${MAGIC_VER} && \
    ./configure --with-tcl=/usr --with-tk=/usr && \
    make -j"$(nproc)" && make install && \
    magic -version && \
    cd / && rm -rf /tmp/build

# --- volare (PDK manager) + sky130 enable ---
RUN mkdir -p "$PDK_ROOT" "$VOLARE_HOME"
RUN python3 -m pip install --no-cache-dir --upgrade pip volare
# enable pinned version into VOLARE_HOME
RUN volare enable --pdk sky130 ${SKY130_VER}

# expose conventional /opt/pdk/sky130A path
RUN ln -s "${VOLARE_HOME}/sky130/${SKY130_VER}/pdk/sky130A" "${PDK_ROOT}/sky130A"

# Helpful env for tools
ENV MAGIC_TECHPATH="$PDK_ROOT/sky130A/libs.tech/magic"
ENV NETGEN_SETUP="$PDK_ROOT/sky130A/libs.tech/netgen/sky130A_setup.tcl"

# --- noVNC supervisor services ---
COPY supervisord.conf /etc/supervisor/conf.d/vnc.conf

EXPOSE 6080
CMD ["/usr/bin/supervisord", "-n"]
