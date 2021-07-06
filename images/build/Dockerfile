#####
## Dockerfile for building tunnelto binaries
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
    cargo build --bins --release
