#####
## Dockerfile for a tunnelto client
##
## https://github.com/agrinman/tunnelto
#####

FROM rust:slim AS build

# Install required packages
RUN apt-get update && \
    apt-get install -y git libssl-dev pkg-config && \
    rm -rf /var/lib/apt/lists/*

ARG TUNNELTO_REPO=https://github.com/agrinman/tunnelto.git
ARG TUNNELTO_TAG=0.1.18
RUN git clone --depth 1 --branch $TUNNELTO_TAG $TUNNELTO_REPO /opt/tunnelto && \
    cd /opt/tunnelto && \
    cargo build --bin tunnelto --release


FROM debian:buster-slim

# Add an unprivileged user to run the tunnelto client
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
    apt-get install -y ca-certificates openssl tini && \
    rm -rf /var/lib/apt/lists/*

# Copy the client binary from the build
COPY --from=build /opt/tunnelto/target/release/tunnelto /usr/bin/
# Copy the entrypoint script
COPY ./tunnelto-init /usr/bin/

USER $TUNNELTO_UID
ENTRYPOINT ["/usr/bin/tini", "-g", "--"]
CMD ["/usr/bin/tunnelto-init"]
