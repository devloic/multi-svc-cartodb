FROM ubuntu:16.04 AS base

ENV PG_MAJOR 10
ENV CARTO_GIS_PPA_FINGERPRINT E54DBDF7355FE6DBEC654BAC6F44D37DD878D6C2

RUN apt-get update \
 && apt-get -y install \
    software-properties-common \
    build-essential \
    pkg-config \
    locales \
    make \
    zip \
    wget \
 && locale-gen en_US.UTF-8 \
 && update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8

# Add all package indexes to the container
COPY ./sources.list.d/*.list /etc/apt/sources.list.d/

# Install PPA keys/fingerprints, then install PPA-sourced packages
RUN apt-key adv --keyserver=keyserver.ubuntu.com --recv-keys $CARTO_GIS_PPA_FINGERPRINT \
 && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - \
 && apt-get update \
 && apt-get install -y --no-install-recommends \
    postgresql-server-dev-$PG_MAJOR \
    gdal-bin \
    libgdal-dev

WORKDIR /carto

COPY ./setup_node_10.sh /carto/setup_node_10.sh

RUN /bin/bash -c /carto/setup_node_10.sh \
 && apt-get update \
 && apt-get install -y nodejs

COPY ./cartodb /carto/cartodb


# Git is used during npm install. We have to re-add the remote to the repo
# because it's installed in the multi-svc-cartodb repo as a submodule, and
# gets quite upset when you copy only the submodule into the container and
# try to work with it absent the parent repo. It stays checked out at the
# revision in the submodule when copied, this just reconnects it to its
# source repository so git functions normally.
RUN apt-get install -y git \
 && rm -rf /carto/cartodb/.git \
 && cd /carto/cartodb \
 && git init \
 && git remote add origin https://github.com/CartoDB/cartodb.git