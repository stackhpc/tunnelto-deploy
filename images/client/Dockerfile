#####
## Dockerfile for a tunnelto client
##
## https://github.com/agrinman/tunnelto
#####

# Naming the build stage like this allows us to have a customisable prefix
ARG TUNNELTO_PREFIX=ghcr.io/stackhpc
FROM ${TUNNELTO_PREFIX}/tunnelto-build as build


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
