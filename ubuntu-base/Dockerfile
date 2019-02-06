# Compile the kernel and its modules for a specific version, with a specific configuration, and a special suffix
FROM KERNEL_BUILDER_IMAGE
ARG KERNEL_VERSION=""
ARG KERNEL_EXTRA=""

# Fetch the latest changes and use the desired kernel version
RUN git fetch --tags && git checkout v${KERNEL_VERSION} && \
    make clean && make mrproper
# Get the desired config, and add the suffix
COPY Kconfig-${KERNEL_VERSION} .config

# Compile the kernel and install the modules to /lib/modules
RUN make EXTRAVERSION=${KERNEL_EXTRA} LOCALVERSION= oldconfig
RUN make EXTRAVERSION=${KERNEL_EXTRA} LOCALVERSION= -j16
RUN make EXTRAVERSION=${KERNEL_EXTRA} LOCALVERSION= modules_install

# Make a tar file of the modules and copy them and the kernel binary and config over to /kernel, which will be used in the next step
RUN mkdir -p /kernel && tar -cf /kernel/modules.tar /lib/modules
RUN cp vmlinux /kernel/vmlinux-${KERNEL_VERSION}${KERNEL_EXTRA} && cp .config /kernel/Kconfig-${KERNEL_VERSION}

# Copy over all artifacts from earlier builds, and extract them into the right places.
FROM scratch
ARG KERNEL_VERSION=""
ARG KERNEL_EXTRA=""
ARG UBUNTU_VERSION=""
ARG MIRROR_PREFIX="fi."
# Add and extract this tar file to populate the whole rootfs of this container
ADD rootfs-${UBUNTU_VERSION}.tar.gz /
# Copy the /kernel directory which was present in the first stage of this multi-stage build and extract the contents
COPY --from=0 /kernel /kernel
RUN tar -xf /kernel/modules.tar -C / --remove-files && \
    mv /kernel/vmlinux-${KERNEL_VERSION}${KERNEL_EXTRA} /boot/ && \
    ln -s /boot/vmlinux-${KERNEL_VERSION}${KERNEL_EXTRA} /boot/vmlinux && \
    mv /kernel/Kconfig-* /boot/

# Set the root password and the default dns server
ARG DNS_SERVER="1.1.1.1"
RUN echo 'root:root' | chpasswd && \
    echo "nameserver ${DNS_SERVER}" > /etc/resolv.conf

# This is the same as "systemctl enable systemd-networkd"
RUN mkdir -p /etc/systemd/system/sockets.target.wants /etc/systemd/system/network-online.target.wants && \
    ln -sf /lib/systemd/system/systemd-networkd.service /etc/systemd/system/dbus-org.freedesktop.network1.service && \
    ln -sf /lib/systemd/system/systemd-networkd.service /etc/systemd/system/multi-user.target.wants/systemd-networkd.service && \
    ln -sf /lib/systemd/system/systemd-networkd.socket /etc/systemd/system/sockets.target.wants/systemd-networkd.socket && \
    ln -sf /lib/systemd/system/systemd-networkd-wait-online.service /etc/systemd/system/network-online.target.wants/systemd-networkd-wait-online.service

# Add the sources.list file so that we can customize from where to get packages
COPY sources.list /etc/apt/
RUN sed -e "s|UBUNTU_CODENAME|$(grep UBUNTU_CODENAME /etc/os-release | cut -d= -f2)|g;s|MIRROR_PREFIX|${MIRROR_PREFIX}|g" -i /etc/apt/sources.list
# Install some utilities, an ssh server and curl, which is needed in most installations
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
