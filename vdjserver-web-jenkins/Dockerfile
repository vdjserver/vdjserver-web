# Base Image
FROM jenkins:2.7.1-alpine

MAINTAINER Walter Scarborough <wscarbor@tacc.utexas.edu>

USER root

RUN apk add --update \
    nodejs \

    && \

    npm install -g mocha-phantomjs \

    && \

    rm -rf /root/.npm \

    && \

    rm -rf /var/cache/apk/*

USER jenkins

COPY docker/plugins/plugins.txt /usr/share/jenkins/plugins.txt
RUN /usr/local/bin/plugins.sh /usr/share/jenkins/plugins.txt
