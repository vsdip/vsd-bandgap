FROM mcr.microsoft.com/devcontainers/base:ubuntu-22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV PDK_ROOT=/opt/pdk
ENV DISPLAY=:1
ENV GEOMETRY=1280x800
ENV DEPTH=24

# 1) Base deps + GUI layer (XFCE + noVNC)
RUN apt-get update && apt-get install -y \
    git build-essential autoconf automake libtool pkg-config \
    tcl tcl-dev tk tk-dev libcairo2-dev libx11-dev libxrender-dev libx11-6 libxaw7-dev \
    python3 python3-venv curl ca-certificates wget sudo \
    xfce4 xfce4-terminal x11vnc xvfb novnc websockify supervisor dbus-x11 \
    # tools (versions are usually good enough for teaching/flows)
    magic ngspice netgen \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

# 2) Build & install open_pdks (Sky130)
#    Note: open_pdks will fetch skywater-pdk and friends as needed.
WORKDIR /tmp/build
RUN git clone https://github.com/RTimothyEdwards/open_pdks.git && \
    cd open_pdks && \
    ./configure --enable-sky130-pdk --with-sky130-variants=all --prefix=${PDK_ROOT} && \
    make -j$(nproc) && \
    make install && \
    rm -rf /tmp/build

# 3) noVNC supervisor services
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY start-vnc.sh /usr/local/bin/start-vnc.sh
RUN chmod +x /usr/local/bin/start-vnc.sh

# 4) Helpful env + defaults
ENV PATH="/usr/local/bin:${PATH}"
ENV MAGIC_TECHPATH="$PDK_ROOT/sky130A/libs.tech/magic"
ENV NGSPICE_PDK="$PDK_ROOT/sky130A"
ENV NETGEN_SETUP="$PDK_ROOT/sky130A/libs.tech/netgen/sky130A_setup.tcl"

EXPOSE 6080
CMD ["/usr/bin/supervisord", "-n"]
