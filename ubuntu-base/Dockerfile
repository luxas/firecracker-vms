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
RUN apt-get update && \
    apt-get -y install --no-install-recommends \
      bash-completion \
      command-not-found \
      openssh-server \
      rng-tools \
      nano \
      curl \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# The handler in this .bash_profile rescans the terminal size after each
# command, so tools like less and htop are actually usable in the VM
COPY .bash_profile /root/
