# translate/pootle:2.5.1
#
# VERSION       0.0.1

FROM debian:wheezy

MAINTAINER Ryan Northey <ryan@synca.io>

ENV DEBIAN_FRONTEND=noninteractive \
    INSTALL_DIR=~/pootle_env \
    POOTLE_PKG='pootle==2.5.1.3'

RUN apt-get update \
    && apt-get install -y \
         build-essential \
         curl \
         libxml2-dev \
         libxslt-dev \
         locales \
         python-dev \
         python-pip \
         sudo \
         zlib1g-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
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
         pootle \
    && sudo -u pootle bash -c "\
         mkdir $INSTALL_DIR \
         && cd $INSTALL_DIR \
         && virtualenv . \
         && . bin/activate \
         && pip install $POOTLE_PKG \
         && pootle init \
         && pootle setup \
         && (pootle runserver --insecure 0.0.0.0:8000 &) \
         && sleep 2 \
         && curl \
                -s \
                -L \
                -o site.html \
              http://127.0.0.1:8000/ \
         && rm site.html \
         && echo 'ALLOWED_HOSTS = [\"*\"]' \
              >> ~/.pootle/pootle.conf"

CMD sudo -u pootle \
      bash -c "\
        cd $INSTALL_DIR \
        && . bin/activate \
        && pootle runserver --insecure 0.0.0.0:8000 "

EXPOSE 8000
