VERSION=18.10
REVISION?=$(shell date --rfc-3339=date)
TAG=${VERSION}-${REVISION}
REGISTRY=luxas
KERNEL_EXTRA?=
KERNEL_BUILDER_IMAGE=luxas/kernel-builder
UID_GID=$(shell id -u):$(shell id -g)
MIRROR_PREFIX?=fi.

# Supported base image versions
ifeq ($(VERSION),18.04)
KERNEL_VERSION=4.14.55
KERNEL_BUILDER_TAG=gcc-5
endif
ifeq ($(VERSION),18.10)
KERNEL_VERSION=4.19.19
KERNEL_BUILDER_TAG=gcc-7
endif

all: build
rootfs-$(VERSION).tar:
	# Ideally, this could be done in the multi-stage Dockerfile. However, there is no support for privileged builds, and
	# debootstrap does a chroot syscall which requires root privileges :/
	# Install a new Ubuntu root filesystem for the desired distro in a temp directory (here /mnt) and create a tar file
	# Use gzip compression for the tarfile to save space, the servers don't have any excess
	docker run -it --rm --privileged -v $(shell pwd):/out ubuntu:${VERSION} /bin/bash -c "\
		source /etc/os-release && \
		apt-get update -y && apt-get install debootstrap -y && \
		debootstrap --arch=amd64 \$$UBUNTU_CODENAME /mnt http://${MIRROR_PREFIX}archive.ubuntu.com/ubuntu/ && \
		cd /mnt && tar -zcf /out/rootfs-${VERSION}.tar.gz . && \
		chown ${UID_GID} /out/rootfs-${VERSION}.tar.gz"

build: rootfs-$(VERSION).tar
	# Build the kernel and the rest of the VM base OS image
	sed -e "s|KERNEL_BUILDER_IMAGE|${KERNEL_BUILDER_IMAGE}:${KERNEL_BUILDER_TAG}|g" Dockerfile | \
		docker build \
			--build-arg KERNEL_VERSION=${KERNEL_VERSION} \
			--build-arg KERNEL_EXTRA=${KERNEL_EXTRA} \
			--build-arg UBUNTU_VERSION=${VERSION} \
			--build-arg MIRROR_PREFIX=${MIRROR_PREFIX} \
			-t ${REGISTRY}/ubuntu-base:${TAG} -f - .
	

push-latest: build-image
	docker tag ${REGISTRY}/ubuntu-base:${TAG} ${REGISTRY}/ubuntu-base:${VERSION}
	docker push ${REGISTRY}/ubuntu-base:${VERSION}
ifeq ($(VERSION),18.10)
	docker tag ${REGISTRY}/ubuntu-base:${TAG} ${REGISTRY}/ubuntu-base:latest
	docker push ${REGISTRY}/ubuntu-base:latest
endif

push-tagged: build-image
	docker push ${REGISTRY}/ubuntu-base:${TAG}

clean:
	rm -rf rootfs-*.tar.gz
