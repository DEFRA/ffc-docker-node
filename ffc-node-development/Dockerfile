ARG REGISTRY=562955126301.dkr.ecr.eu-west-2.amazonaws.com
ARG BASE_VERSION=1.0.0
ARG PARENT_IMAGE=ffc-node-parent
FROM $REGISTRY/$PARENT_IMAGE:$BASE_VERSION

ENV NODE_ENV development

USER root
# node-gyp is a common requirement for NPM packages. Install it here.
RUN apk update && \
    apk add --no-cache git=2.24.1-r0 && \
    apk add --no-cache --virtual .gyp python=2.7.16-r3 make=4.2.1-r2 g++=9.2.0-r3

USER node

