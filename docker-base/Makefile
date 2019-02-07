VERSION?=18.10
REVISION?=$(shell date --rfc-3339=date)
TAG=${VERSION}-${REVISION}
REGISTRY?=luxas
UID_GID=$(shell id -u):$(shell id -g)

all: build
build:
	sed -e "s|UBUNTU_VERSION|${VERSION}|g" Dockerfile | \
		docker build -t ${REGISTRY}/docker-base:${TAG} .

push-latest:
	docker tag ${REGISTRY}/docker-base:${TAG} ${REGISTRY}/docker-base:${VERSION}
	docker push ${REGISTRY}/docker-base:${VERSION}
ifeq ($(VERSION),18.10)
	docker tag ${REGISTRY}/docker-base:${TAG} ${REGISTRY}/docker-base:latest
	docker push ${REGISTRY}/docker-base:latest
endif

push-tagged:
	docker push ${REGISTRY}/docker-base:${TAG}
