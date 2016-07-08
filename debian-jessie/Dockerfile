# translate/pootle:debian-jessie
#
# VERSION       0.0.1

FROM debian:jessie

MAINTAINER Ryan Northey <ryan@synca.io>

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive \
         apt-get install -y \
           build-essential \
           coreutils \
           curl \
           git \
           libxml2-dev \
           libxslt-dev \
           locales \
           python-pip \
           python-dev \
           redis-server \
           sudo \
           zlib1g-dev \
    && apt-get clean \
    && echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
    && locale-gen \
    && update-locale LANG=en_US.UTF-8 \
    && pip install virtualenv \
    && groupadd -r pootle \
    && useradd \
           -m \
           -d /home/pootle \
           -k /etc/skel \
           -s /bin/bash \
           -g pootle \
         pootle
