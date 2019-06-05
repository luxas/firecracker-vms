VERSION?=18.04
REVISION?=$(shell date --rfc-3339=date)
TAG=${VERSION}-${REVISION}
REGISTRY?=luxas
KERNEL_EXTRA?=
KERNEL_BUILDER_IMAGE=luxas/kernel-builder:gcc-7
UID_GID=$(shell id -u):$(shell id -g)
MIRROR_PREFIX?=fi.

# Supported base image versions
ifeq ($(VERSION),18.04)
KERNEL_VERSION=4.15
KERNEL_EXTRA=-50
endif
ifeq ($(VERSION),18.10)
KERNEL_VERSION=4.19.47
endif
ifeq ($(VERSION),19.04)
KERNEL_VERSION=5.1
endif

all: build
rootfs: build/rootfs-$(VERSION).tar.gz
build/rootfs-$(VERSION).tar.gz:
	# Create a new Ubuntu filesystem from scratch using Debootstrap
	docker run -it --rm --privileged -v $(shell pwd)/build:/build ubuntu:${VERSION} /bin/bash -c "\
		source /etc/os-release && \
		apt-get update -y && apt-get install debootstrap -y && \
		debootstrap --arch=amd64 \$$UBUNTU_CODENAME /mnt http://${MIRROR_PREFIX}archive.ubuntu.com/ubuntu/ && \
		cd /mnt && tar -czf /build/rootfs-${VERSION}.tar.gz . && \
		chown ${UID_GID} /build/rootfs-${VERSION}.tar.gz"

kernel: build/kernel-$(KERNEL_VERSION)$(KERNEL_EXTRA).tar.gz
build/kernel-$(KERNEL_VERSION)$(KERNEL_EXTRA).tar.gz:
	# Compile the kernel from the specified config, and generate a tarball with the artifacts
	docker run -it \
		-v $(shell pwd)/build:/build \
		-v $(shell pwd)/Kconfig-${KERNEL_VERSION}:/tmp/.config \
		${KERNEL_BUILDER_IMAGE} /bin/bash -c "\
			git fetch --tags && git checkout v${KERNEL_VERSION} && \
			make clean && make mrproper && cp /tmp/.config . && \
			make EXTRAVERSION=${KERNEL_EXTRA} LOCALVERSION= olddefconfig && \
			make EXTRAVERSION=${KERNEL_EXTRA} LOCALVERSION= oldconfig && \
			make EXTRAVERSION=${KERNEL_EXTRA} LOCALVERSION= -j32 && \
			make EXTRAVERSION=${KERNEL_EXTRA} LOCALVERSION= modules_install && \
			cp vmlinux /boot/vmlinux-${KERNEL_VERSION}${KERNEL_EXTRA} && \
			ln -s /boot/vmlinux-${KERNEL_VERSION}${KERNEL_EXTRA} /boot/vmlinux && \
			cp .config /boot/Kconfig-${KERNEL_VERSION}${KERNEL_EXTRA} && \
			tar -czf /build/kernel-${KERNEL_VERSION}${KERNEL_EXTRA}.tar.gz /boot /lib/modules && \
			chown ${UID_GID} /build/kernel-${KERNEL_VERSION}${KERNEL_EXTRA}.tar.gz"

kshell:
	docker run -it \
		-v $(shell pwd)/build:/build \
		-v $(shell pwd)/Kconfig-${KERNEL_VERSION}:/tmp/.config \
		${KERNEL_BUILDER_IMAGE} /bin/bash

update-kconfig:
	docker run -it \
                -v $(shell pwd)/build:/build \
                -v $(shell pwd)/Kconfig-${KERNEL_VERSION}:/tmp/.config \
                ${KERNEL_BUILDER_IMAGE} /bin/bash -c "\
			git fetch --tags && git checkout v${KERNEL_VERSION} && \
			make clean && make mrproper && cp /tmp/.config . && \
			make EXTRAVERSION=${KERNEL_EXTRA} LOCALVERSION= olddefconfig && \
			cp .config /tmp/.config"

build: rootfs kernel
	# Build the kernel and the rest of the VM base OS image
	docker build \
		--build-arg KERNEL_VERSION=${KERNEL_VERSION} \
		--build-arg KERNEL_EXTRA=${KERNEL_EXTRA} \
		--build-arg UBUNTU_VERSION=${VERSION} \
		--build-arg MIRROR_PREFIX=${MIRROR_PREFIX} \
		-t ${REGISTRY}/ubuntu-base:${TAG} .

push-latest:
	docker tag ${REGISTRY}/ubuntu-base:${TAG} ${REGISTRY}/ubuntu-base:${VERSION}
	docker push ${REGISTRY}/ubuntu-base:${VERSION}
ifeq ($(VERSION),18.04)
	docker tag ${REGISTRY}/ubuntu-base:${TAG} ${REGISTRY}/ubuntu-base:latest
	docker push ${REGISTRY}/ubuntu-base:latest
endif

push-tagged:
	docker push ${REGISTRY}/ubuntu-base:${TAG}

clean:
	rm -rf rootfs-*.tar.gz
