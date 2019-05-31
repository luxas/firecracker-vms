# Copy over all artifacts from earlier builds, and extract them into the right places.
FROM scratch
ARG KERNEL_VERSION=""
ARG KERNEL_EXTRA=""
ARG UBUNTU_VERSION=""
ARG MIRROR_PREFIX="fi."
# Add and extract this tar file to populate the whole rootfs of this container
ADD build/rootfs-${UBUNTU_VERSION}.tar.gz /
ADD build/kernel-${KERNEL_VERSION}${KERNEL_EXTRA}.tar.gz /

# Set the root password to root
RUN echo 'root:root' | chpasswd

# This is the same as "systemctl enable systemd-networkd && systemctl enable use-systemd-resolved"
# use-systemd-resolved.service is here to force /etc/resolv.conf being a symlink to the systemd-resolved-managed file at runtime
COPY use-systemd-resolved.service /etc/systemd/system/
RUN mkdir -p /etc/systemd/system/sockets.target.wants /etc/systemd/system/network-online.target.wants && \
    ln -sf /lib/systemd/system/systemd-networkd.service /etc/systemd/system/dbus-org.freedesktop.network1.service && \
    ln -sf /lib/systemd/system/systemd-networkd.service /etc/systemd/system/multi-user.target.wants/systemd-networkd.service && \
    ln -sf /lib/systemd/system/systemd-networkd.socket /etc/systemd/system/sockets.target.wants/systemd-networkd.socket && \
    ln -sf /lib/systemd/system/systemd-networkd-wait-online.service /etc/systemd/system/network-online.target.wants/systemd-networkd-wait-online.service && \
    ln -sf /etc/systemd/system/use-systemd-resolved.service /etc/systemd/system/multi-user.target.wants/use-systemd-resolved.service

# Add the sources.list file so that we can customize from where to get packages
COPY sources.list /etc/apt/
RUN sed -e "s|UBUNTU_CODENAME|$(grep UBUNTU_CODENAME /etc/os-release | cut -d= -f2)|g;s|MIRROR_PREFIX|${MIRROR_PREFIX}|g" -i /etc/apt/sources.list
# Install some utilities, which are needed in most installations
# PolicyKit is required for systemd-networkd to set the transient hostname
RUN apt-get update && \
    apt-get -y install --no-install-recommends \
      policykit-1 \
      bash-completion \
      command-not-found \
      openssh-server \
      rng-tools \
      vim \
      curl \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# The handler in this .bash_profile rescans the terminal size after each
# command, so tools like less and htop are actually usable in the VM
COPY .bash_profile /root/

# Add the systemd-networkd config file for enabling DHCP on eth0
COPY 20-dhcp.network /etc/systemd/networkd/

# Enable the node exporter by default listening on 0.0.0.0:9100
# If you don't want this service to automatically start, you can opt out with
# rm /etc/systemd/system/multi-user.target.wants/node_exporter.service
COPY node_exporter.service /etc/systemd/system/
ENV NODE_EXPORTER_VERSION=0.17.0
RUN curl -sSL https://github.com/prometheus/node_exporter/releases/download/v${NODE_EXPORTER_VERSION}/node_exporter-${NODE_EXPORTER_VERSION}.linux-amd64.tar.gz > /tmp/node_exporter.tar.gz && \
    echo "d2e00d805dbfdc67e7291ce2d2ff151f758dd7401dd993411ff3818d0e231489  /tmp/node_exporter.tar.gz" | sha256sum -c && \
    tar -xzf /tmp/node_exporter.tar.gz --strip-components=1 -C /usr/bin node_exporter-${NODE_EXPORTER_VERSION}.linux-amd64/node_exporter && \
    ln -sf /etc/systemd/system/node_exporter.service /etc/systemd/system/multi-user.target.wants/node_exporter.service
