FROM alpine:3.3
MAINTAINER Bastiaan Bakker <bastiaan@nakednerds.net>

# This is the release of Nomad to pull in.
ENV NOMAD_VERSION=0.3.2

# This is the release of https://github.com/hashicorp/docker-base to pull in order
# to provide HashiCorp-built versions of basic utilities like dumb-init and gosu.
ENV DOCKER_BASE_VERSION=0.0.4

# Create a nomad user and group first so the IDs get set the same way, even as
# the rest of this may change over time.
RUN addgroup nomad && \
    adduser -S -G nomad nomad

# Set up certificates, our base tools, and Nomad.
RUN apk add --no-cache ca-certificates gnupg && \
    gpg --recv-keys 91A6E7F85D05C65630BEF18951852D87348FFC4C && \
    mkdir -p /tmp/build && \
    cd /tmp/build && \
    wget https://releases.hashicorp.com/docker-base/${DOCKER_BASE_VERSION}/docker-base_${DOCKER_BASE_VERSION}_linux_amd64.zip && \
    wget https://releases.hashicorp.com/docker-base/${DOCKER_BASE_VERSION}/docker-base_${DOCKER_BASE_VERSION}_SHA256SUMS && \
    wget https://releases.hashicorp.com/docker-base/${DOCKER_BASE_VERSION}/docker-base_${DOCKER_BASE_VERSION}_SHA256SUMS.sig && \
    gpg --batch --verify docker-base_${DOCKER_BASE_VERSION}_SHA256SUMS.sig docker-base_${DOCKER_BASE_VERSION}_SHA256SUMS && \
    grep ${DOCKER_BASE_VERSION}_linux_amd64.zip docker-base_${DOCKER_BASE_VERSION}_SHA256SUMS | sha256sum -c && \
    unzip docker-base_${DOCKER_BASE_VERSION}_linux_amd64.zip && \
    cp bin/gosu bin/dumb-init /bin && \
    wget https://releases.hashicorp.com/nomad/${NOMAD_VERSION}/nomad_${NOMAD_VERSION}_linux_amd64.zip && \
    wget https://releases.hashicorp.com/nomad/${NOMAD_VERSION}/nomad_${NOMAD_VERSION}_SHA256SUMS && \
    wget https://releases.hashicorp.com/nomad/${NOMAD_VERSION}/nomad_${NOMAD_VERSION}_SHA256SUMS.sig && \
    gpg --batch --verify nomad_${NOMAD_VERSION}_SHA256SUMS.sig nomad_${NOMAD_VERSION}_SHA256SUMS && \
    grep nomad_${NOMAD_VERSION}_linux_amd64.zip nomad_${NOMAD_VERSION}_SHA256SUMS | sha256sum -c && \
    unzip -d /bin nomad_${NOMAD_VERSION}_linux_amd64.zip && \
    cd /tmp && \
    rm -rf /tmp/build && \
    apk del gnupg && \
    rm -rf /root/.gnupg

# The /nomad/data dir is used by nomad to store state. The agent will be started
# with /nomad/config as the configuration directory so you can add additional
# config files in that location.
RUN mkdir -p /nomad/data && \
    mkdir -p /nomad/config && \
    chown -R nomad:nomad /nomad

# Expose the nomad data directory as a volume since there's mutable state in there.
VOLUME /nomad/data

# The port used to run the HTTP server. Applies to both client and server nodes.
EXPOSE 4646

# The port used for internal RPC communication between agents and servers, and for
# inter-server traffic for the consensus algorithm (raft). Only used on server nodes.
EXPOSE 4647

# The port used for the gossip protocol for cluster membership. Both TCP and UDP
# should be routable between the server nodes on this port. Only used on server nodes.
EXPOSE 4648 4648/udp

# The entry point script uses dumb-init as the top-level
# process to reap any zombie processes created by nomad sub-processes.
COPY docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh
ENTRYPOINT ["docker-entrypoint.sh"]

# By default you'll get a single-node development server that bootstraps itself.
# Don't use this configuration for production.
CMD ["agent", "-dev"]
