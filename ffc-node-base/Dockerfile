ARG NODE_VERSION=10.18.0
FROM node:$NODE_VERSION-alpine
ARG NODE_VERSION
ARG VERSION=1.0.0

RUN apk update && apk add --no-cache tini=0.18.0-r0
ENTRYPOINT ["/sbin/tini", "--"]

USER node
WORKDIR /home/node

ENV NODE_ENV production
ENV NPM_CONFIG_PREFIX=/home/node/.npm-global
ENV PATH=$PATH:/home/node/.npm-global/bin
LABEL uk.gov.defra.ffc-node-base.version=$VERSION \
      uk.gov.defra.ffc-node-base.node-version=$NODE_VERSION

# npm 6.13.4 comes preinstalled in node:10.18.0-alpine. Adding this command ensures that version is the one in use, but adds 20mb to the image size
#RUN npm install -g npm@6.13.4
