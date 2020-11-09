#
# Copyright (c) 2019, 2020 Oracle and/or its affiliates. All rights reserved.
# Licensed under the Universal Permissive License v 1.0 as shown at http://oss.oracle.com/licenses/upl.
# 

# Oracle Instant Client version
ARG oracleClientVersion=19.8

###############################
#  ----- Image Assets -----   #
###############################
FROM node:12-alpine as assets-builder

RUN apk update && apk add --no-cache \
    autoconf \
    automake \
    bash \
    g++ \
    libtool \
    libc6-compat \
    libjpeg-turbo-dev \
    libpng-dev \
    make \
    nasm

COPY src/assets/package.json /tmp/package.json
COPY src/assets/package-lock.json /tmp/package-lock.json
RUN cd /tmp && npm ci
RUN mkdir -p /app/assets && cp -a /tmp/node_modules /app/assets/
RUN rm -rf /tmp/node_modules

WORKDIR /app/assets
COPY src/assets/. .
RUN rm -rf /app/assets/hero

RUN npm run build
RUN rm -rf node_modules products hero *.md
#  ----- Image Assets -----   #

###############################
#    ----- Base Image ------  #
#      Everything needed      #
###############################

# Base image with Oracle Instant Client Basic Lite
FROM oraclelinux:7-slim AS base
ARG oracleClientVersion
RUN yum clean metadata && \
  yum update -y && \
  yum -y install oracle-release-el7 && \
  yum-config-manager --enable ol7_oracle_instantclient && \
  yum-config-manager --enable ol7_latest && \
  yum -y install oracle-instantclient${oracleClientVersion}-basiclite && \
  yum -y install zip && \
  yum clean all && \
  rm -rf /var/cache/yum

#    ----- Base Image ------  #

###############################
#  ----- Runtime Image -----  #
#   runtime app and stack     #
###############################

# Runtime Image
FROM base
ARG oracleClientVersion
WORKDIR /

# Create ORM package
COPY deploy/basic/VERSION /
WORKDIR /basic
COPY deploy/basic/terraform/*.tf /basic/
COPY deploy/basic/terraform/*.tfvars.example /basic/
COPY deploy/basic/terraform/schema.yaml /basic/
COPY deploy/basic/terraform/scripts /basic/scripts
COPY src/catalogue/dbdata/atp_mushop_catalogue.sql /basic/scripts/
COPY --from=assets-builder /app/assets/dist /basic/images
RUN mkdir /package && zip -r /package/mushop-basic-stack.zip .

VOLUME ["/usr/lib/oracle/${oracleClientVersion}/client64/lib/network/admin/"]
VOLUME ["/transfer/"]

#    ----- Base Image ------  #
