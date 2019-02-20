FROM luxas/ubuntu-base:UBUNTU_VERSION

RUN apt-get update && \
    apt-get -y install --no-install-recommends \
      docker.io \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

ENV DOCKER_COMPOSE_VERSION=1.23.2
RUN curl -sSL https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-Linux-x86_64 > /usr/local/bin/docker-compose && \
    chmod +x /usr/local/bin/docker-compose

COPY node_exporter.service /etc/systemd/system/
ENV NODE_EXPORTER_VERSION=0.17.0
RUN curl -sSL https://github.com/prometheus/node_exporter/releases/download/v${NODE_EXPORTER_VERSION}/node_exporter-${NODE_EXPORTER_VERSION}.linux-amd64.tar.gz > /tmp/node_exporter.tar.gz && \
    echo "d2e00d805dbfdc67e7291ce2d2ff151f758dd7401dd993411ff3818d0e231489  /tmp/node_exporter.tar.gz" | sha256sum -c && \
    tar -xzf /tmp/node_exporter.tar.gz --strip-components=1 -C /usr/bin node_exporter-${NODE_EXPORTER_VERSION}.linux-amd64/node_exporter && \
    ln -sf /etc/systemd/system/node_exporter.service /etc/systemd/system/multi-user.target.wants/node_exporter.service
