# translate/pootle:2.5.1-postgres
#
# VERSION       0.0.1

FROM debian:wheezy

MAINTAINER Ryan Northey <ryan@synca.io>

ENV DEBIAN_FRONTEND=noninteractive \
    INSTALL_DIR=~/pootle_env \
    POOTLE_PKG='pootle==2.5.1.3'

RUN apt-get update -qq \
    && apt-get install -y \
         build-essential \
         curl \
         libmysqlclient-dev \
         libxml2-dev \
         libxslt-dev \
         locales \
         mysql-server \
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
    && /etc/init.d/mysql start \
    && mysql -e "\
         CREATE DATABASE pootledb \
           CHARACTER SET utf8 \
           DEFAULT COLLATE utf8_general_ci; \
         GRANT ALL PRIVILEGES ON pootledb.* \
           TO pootle@localhost \
           IDENTIFIED BY 'CHANGEME'; \
         FLUSH PRIVILEGES;" \
    && sudo -u pootle bash -c "\
         mkdir $INSTALL_DIR \
         && cd $INSTALL_DIR \
         && virtualenv . \
         && . bin/activate \
         && pip install \
              $POOTLE_PKG \
              MySQL-python \
         && pootle init \
         && echo 'DATABASES[\"default\"][\"ENGINE\"] = \"django.db.backends.mysql\"' \
              >> ~/.pootle/pootle.conf \
         && echo 'DATABASES[\"default\"][\"NAME\"] = \"pootledb\"' \
              >> ~/.pootle/pootle.conf \
         && echo 'DATABASES[\"default\"][\"USER\"] = \"pootle\"' \
              >> ~/.pootle/pootle.conf \
         && echo 'DATABASES[\"default\"][\"PASSWORD\"] = \"CHANGEME\"' \
              >> ~/.pootle/pootle.conf \
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
              >> ~/.pootle/pootle.conf" \
    && /etc/init.d/mysql stop

CMD /etc/init.d/mysql start \
    && sudo -u pootle \
         bash -c "\
           cd $INSTALL_DIR \
           && . bin/activate \
           && pootle runserver --insecure 0.0.0.0:8000 "

EXPOSE 8000
