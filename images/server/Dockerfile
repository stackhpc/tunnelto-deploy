#####
## Dockerfile for a tunnelto server
##
## https://github.com/agrinman/tunnelto
#####

FROM rust:slim AS build

# Install required packages
RUN apt-get update && \
    apt-get install -y git libssl-dev pkg-config && \
    rm -rf /var/lib/apt/lists/*

# Add the patches
# WARNING: THE PATCHES WILL LIKELY NEED TO CHANGE WHEN UPGRADING TUNNELTO
COPY ./patch /opt/tunnelto/patch/

ARG TUNNELTO_REPO=https://github.com/agrinman/tunnelto.git
ARG TUNNELTO_TAG=0.1.18
# At the time of writing, auth is not configurable via the config
# There is an open PR to fix this, and add sqlite as a store
#   https://github.com/agrinman/tunnelto/pull/50
# For now, we need to manually patch main.rs after cloning
RUN git clone --depth 1 --branch $TUNNELTO_TAG $TUNNELTO_REPO /opt/tunnelto/src && \
    patch /opt/tunnelto/src/tunnelto_server/src/main.rs /opt/tunnelto/patch/main.rs.patch && \
    patch /opt/tunnelto/src/tunnelto_server/src/auth/mod.rs /opt/tunnelto/patch/auth.mod.rs.patch && \
    cd /opt/tunnelto/src && \
    cargo build --bin tunnelto_server --release


FROM debian:buster-slim

# Add an unprivileged user to run the tunnelto server
ENV TUNNELTO_UID 1001
ENV TUNNELTO_GID 1001
ENV TUNNELTO_USER tunnelto
ENV TUNNELTO_GROUP tunnelto
RUN groupadd --gid $TUNNELTO_GID $TUNNELTO_GROUP && \
    useradd \
      --no-create-home \
      --no-user-group \
      --shell /sbin/nologin \
      --gid $TUNNELTO_GID \
      --uid $TUNNELTO_UID \
      $TUNNELTO_USER

RUN apt-get update && \
    apt-get install -y openssl tini && \
    rm -rf /var/lib/apt/lists/*

# Copy the server binary from the build
COPY --from=build /opt/tunnelto/src/target/release/tunnelto_server /usr/bin/

# HTTP clients connect via this port to access services
EXPOSE 8080
# tunnelto clients connect via this port to begin tunnels
EXPOSE 5000
# Other tunnelto servers connect via this port for the gossip protocol
EXPOSE 6000

USER $TUNNELTO_UID
ENTRYPOINT ["/usr/bin/tini", "-g", "--"]
CMD ["/usr/bin/tunnelto_server"]
