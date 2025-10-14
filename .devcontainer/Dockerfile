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

# --- Desktop + tools omitted for brevity above ---

# Put PDK_ROOT BEFORE using volare so it installs under /opt/pdk
ENV PDK_ROOT=/opt/pdk
RUN mkdir -p "$PDK_ROOT"

# Install volare and enable a known-good sky130 version (pin for reproducibility)
RUN pip install --no-cache-dir volare
# (This hash is exactly what your logs showed as enabled and known-good)
RUN volare enable --pdk sky130 c6d73a35f524070e85faff4a6a9eef49553ebc2b

# Create conventional /opt/pdk/sky130A symlink to the enabled tree
RUN set -eux; \
    VOLARE_ROOT="$(volare ls --pdk sky130 | head -n1)"; \
    echo "Volare PDK root: ${VOLARE_ROOT}"; \
    TARGET="$(find "${VOLARE_ROOT}" -type d -path '*/pdk/sky130A' | head -n1)"; \
    echo "Resolved sky130A path: ${TARGET}"; \
    test -d "${TARGET}"; \
    ln -s "${TARGET}" "${PDK_ROOT}/sky130A"

# Helpful env for the tools
ENV MAGIC_TECHPATH="$PDK_ROOT/sky130A/libs.tech/magic"
ENV NETGEN_SETUP="$PDK_ROOT/sky130A/libs.tech/netgen/sky130A_setup.tcl"

# noVNC supervisor files (keep your existing ones)
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY start-vnc.sh /usr/local/bin/start-vnc.sh
RUN chmod +x /usr/local/bin/start-vnc.sh

EXPOSE 6080
CMD ["/usr/bin/supervisord", "-n"]

