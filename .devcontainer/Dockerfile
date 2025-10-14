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

# --- Get prebuilt Sky130 via volare (no compile, small disk) ---
RUN pip install --no-cache-dir volare

# Pick the latest prebuilt sky130 version at build time and enable it
# You can pin later by replacing the awk logic with a fixed hash.
RUN set -eux; \
    LATEST="$(volare ls-remote --pdk sky130 | awk 'NR==1{print $1}')" ; \
    echo "Enabling sky130 version: ${LATEST}"; \
    volare enable --pdk sky130 "${LATEST}"

# Create conventional PDK_ROOT layout & symlink to the enabled sky130A path
ENV PDK_ROOT=/opt/pdk
RUN set -eux; \
    mkdir -p "${PDK_ROOT}"; \
    TARGET="$(find /root/.volare -type d -path '*/sky130/*/pdk/sky130A' | head -n1)"; \
    echo "Resolved sky130A path: ${TARGET}"; \
    test -d "${TARGET}"; \
    ln -s "${TARGET}" "${PDK_ROOT}/sky130A"

# Helpful env for tools
ENV MAGIC_TECHPATH="$PDK_ROOT/sky130A/libs.tech/magic"
ENV NETGEN_SETUP="$PDK_ROOT/sky130A/libs.tech/netgen/sky130A_setup.tcl"

# noVNC services
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY start-vnc.sh /usr/local/bin/start-vnc.sh
RUN chmod +x /usr/local/bin/start-vnc.sh

EXPOSE 6080
CMD ["/usr/bin/supervisord", "-n"]
