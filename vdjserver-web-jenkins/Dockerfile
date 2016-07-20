# Base Image
FROM jenkins:2.7.1

MAINTAINER Walter Scarborough <wscarbor@tacc.utexas.edu>

USER root

RUN apt-get update && apt-get install -y \
    make \
    wget \
    xz-utils

ENV NODE_VERSION=4.4.7
RUN cd /root \
    && wget https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz \
    && ls \
    && tar -xf node-v$NODE_VERSION-linux-x64.tar.xz \
    && cp node-v$NODE_VERSION-linux-x64/bin/node /usr/local/bin \
    && cp -R node-v$NODE_VERSION-linux-x64/lib/node_modules /usr/local/lib \
    && ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm \
    && rm -R /root/node-v$NODE_VERSION-linux-x64

RUN npm install -g \
    mocha-phantomjs \
    phantomjs

USER jenkins

COPY docker/plugins/plugins.txt /usr/share/jenkins/plugins.txt
RUN /usr/local/bin/plugins.sh /usr/share/jenkins/plugins.txt
