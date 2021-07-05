#####
## Dockerfile for building tunnelto binaries
##
## https://github.com/agrinman/tunnelto
#####

FROM rust:slim

# Install required packages
RUN apt-get update && \
    apt-get install -y git libssl-dev pkg-config && \
    rm -rf /var/lib/apt/lists/*

ARG TUNNELTO_REPO=https://github.com/agrinman/tunnelto.git
ARG TUNNELTO_TAG=0.1.18
RUN git clone --depth 1 --branch $TUNNELTO_TAG $TUNNELTO_REPO /opt/tunnelto && \
    cd /opt/tunnelto && \
    cargo build --bins --release
