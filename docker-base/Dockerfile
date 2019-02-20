FROM luxas/ubuntu-base:UBUNTU_VERSION

RUN apt-get update && \
    apt-get -y install --no-install-recommends \
      docker.io \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

ENV DOCKER_COMPOSE_VERSION=1.23.2
RUN curl -sSL https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-Linux-x86_64 > /usr/local/bin/docker-compose && \
    chmod +x /usr/local/bin/docker-compose
