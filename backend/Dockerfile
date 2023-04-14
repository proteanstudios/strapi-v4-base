# create base image with all packages, proper entrypoint, and directories created
FROM node:18-alpine AS packages

# install any packages we need from apt here
RUN apk add --update dumb-init curl mysql-client

# using the base image, create an image containing all of our files
# and dependencies installed, devDeps and test directory included
FROM node:18-alpine AS dependencies

# Installing libvips-dev for sharp Compatibility
# RUN apk add --update build-base gcc autoconf automake zlib-dev libpng-dev nasm bash vips-dev

#this is just to build the spa app
ARG NODE_ENV=development
ENV NODE_ENV=${NODE_ENV}

# all of our code will live in `/app`
WORKDIR /usr/src/app

COPY . .

# Instance correct Sharp lib 
# Installing libvips-dev for sharp Compatibility
RUN apk update && apk add --no-cache build-base gcc autoconf automake zlib-dev libpng-dev nasm bash vips-dev
RUN npm install --platform=linuxmusl --arch=x64 sharp
RUN npm install mysql2

RUN npm set progress=false \
    && npm config set depth 0 \
    && npm i

# test running image using all of the files and devDeps
FROM node:18-alpine AS test

USER node

WORKDIR /usr/src/app

COPY --from=packages /usr/bin/dumb-init /usr/bin/dumb-init
COPY --from=dependencies --chown=node:node /usr/src/app . 

ENV NODE_ENV=test

CMD ["dumb-init", "npx", "strapi", "test"]

# dev ready image, contains devDeps for test running and debugging
FROM node:18-alpine AS development

USER node

WORKDIR /usr/src/app

COPY --from=packages /usr/bin/dumb-init /usr/bin/dumb-init
# COPY --from=packages /usr/bin/mysql /usr/bin/mysql
COPY --from=dependencies --chown=node:node /usr/src/app ./

ENV NODE_ENV=development
# expose a port from the container to the outside world
# this is the default port defined in server/manifest.js, and
# will need to be updated if you change the default port
EXPOSE 1337
CMD ["dumb-init",  "npx", "strapi", "develop"]

# release ready image, devDeps are pruned and tests removed for size control
FROM node:18-alpine AS release

USER node

WORKDIR /usr/src/app

COPY --from=packages /usr/bin/dumb-init /usr/bin/dumb-init
COPY --from=packages /usr/bin/mysql /usr/bin/mysql
COPY --from=dependencies --chown=node:node /usr/src/app . 

ENV NODE_ENV=production

# prune non-prod dependencies, remove test files
RUN npm prune --production \
    && rm -rf ./test
# expose a port from the container to the outside world
# this is the default port defined in server/manifest.js, and
# will need to be updated if you change the default port
EXPOSE 1337
CMD ["dumb-init",  "npx", "strapi", "start"]
