FROM mcr.microsoft.com/devcontainers/base:ubuntu-22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV PDK_ROOT=/opt/pdk
ENV DISPLAY=:1
ENV GEOMETRY=1280x800
ENV DEPTH=24

# --- Desktop + VNC-in-browser + basic tools (small, stable) ---
RUN apt-get update && apt-get install -y \
    git curl ca-certificates wget sudo \
    python3 python3-pip \
    xfce4 xfce4-terminal x11vnc xvfb novnc websockify supervisor dbus-x11 \
    magic ngspice netgen \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# --- Get prebuilt Sky130A via volare (saves space/time vs building) ---
RUN pip install --no-cache-dir volare

# Choose a widely-used Sky130A release; you can change version later with `volare ls-remote`
# This downloads into /root/.volare and creates the local store.
RUN volare enable --pdk sky130a --version latest

# Make conventional PDK_ROOT layout & symlink
RUN mkdir -p ${PDK_ROOT} && \
    ln -s $(volare show --pdk sky130a --path) ${PDK_ROOT}/sky130A

# Helpful env for tools
ENV MAGIC_TECHPATH="$PDK_ROOT/sky130A/libs.tech/magic"
ENV NETGEN_SETUP="$PDK_ROOT/sky130A/libs.tech/netgen/sky130A_setup.tcl"

# noVNC services
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY start-vnc.sh /usr/local/bin/start-vnc.sh
RUN chmod +x /usr/local/bin/start-vnc.sh

EXPOSE 6080
CMD ["/usr/bin/supervisord", "-n"]
