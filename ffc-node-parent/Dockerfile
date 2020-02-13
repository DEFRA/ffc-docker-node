ARG NODE_VERSION=10.18.0
FROM node:$NODE_VERSION-alpine AS production
ARG NODE_VERSION
ARG VERSION=1.0.0

# We need a basic init process to handle signals and reap zombie processes, tini handles that
RUN apk update && apk add --no-cache tini=0.18.0-r0
ENTRYPOINT ["/sbin/tini", "--"]

# Never run as root, use the node user from the alpine image
USER node
# Default workdir for all the containers that use this image
WORKDIR /home/node

ENV NODE_ENV production
# Set global npm dependancies to be stored under the node user directory
ENV NPM_CONFIG_PREFIX=/home/node/.npm-global
ENV PATH=$PATH:/home/node/.npm-global/bin
LABEL uk.gov.defra.ffc-node-parent.version=$VERSION \
      uk.gov.defra.ffc-node-parent.node-version=$NODE_VERSION

FROM production AS development

ENV NODE_ENV development

USER root
# node-gyp is a common requirement for NPM packages. Install it here.
RUN apk update && \
    apk add --no-cache git=2.24.1-r0 && \
    apk add --no-cache --virtual .gyp python=2.7.16-r3 make=4.2.1-r2 g++=9.2.0-r3

USER node
