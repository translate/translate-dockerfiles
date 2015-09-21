# translate/pootle:2.7-dev
#
# VERSION       0.0.1

FROM debian:jessie

MAINTAINER Ryan Northey <ryan@synca.io>

ENV DEBIAN_FRONTEND=noninteractive \
    INSTALL_DIR=~/pootle_env \
    POOTLE_PKG='-e git://github.com/translate/pootle#egg=pootle'

ENV POOTLE_LOG_DIR="$INSTALL_DIR/var/logs" \
    POOTLE_DB_DIR="$INSTALL_DIR/var/db"

RUN apt-get update \
    && apt-get install -y \
         build-essential \
         coreutils \
         curl \
         git \
         libjpeg-dev \
         libfreetype6-dev \
         liblcms2-dev \
         libmysqlclient-dev \
         libtiff5-dev \
         libwebp-dev \
         libxml2-dev \
         libxslt-dev \
         locales \
         mysql-server \
         postgresql-9.4 \
         postgresql-server-dev-9.4 \
         python-pip \
         python-dev \
         nodejs \
         npm \
         redis-server \
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
    && update-alternatives --install \
         /usr/bin/node node /usr/bin/nodejs 99 \
    && /etc/init.d/mysql start \
    && mysql -e "\
         CREATE DATABASE pootledb \
           CHARACTER SET utf8 \
           DEFAULT COLLATE utf8_general_ci; \
         GRANT ALL PRIVILEGES ON pootledb.* \
           TO pootle@localhost \
           IDENTIFIED BY 'CHANGEME'; \
         FLUSH PRIVILEGES;" \
    && /etc/init.d/postgresql start \
    && sudo -u postgres bash -c "\
         psql -c \"CREATE USER pootle WITH PASSWORD 'CHANGEME';\" \
         && createdb \
                --encoding='utf-8' \
                --locale=en_US.utf8 \
                --template=template0 \
                --owner=pootle \
              pootledb" \
    && /etc/init.d/redis-server start \
    && sudo -u pootle \
         bash -c "\
           mkdir -p $INSTALL_DIR \
           && cd $INSTALL_DIR \
           && virtualenv . \
           && . bin/activate \
           && mkdir -p ~/.pootle \
           && mkdir -p $POOTLE_LOG_DIR \
           && mkdir -p $POOTLE_DB_DIR \
           && pip install \
                MySQL-python \
                psycopg2 \
                $POOTLE_PKG \
           && pootle init \
                  --db=mysql \
           && echo 'DATABASES[\"default\"][\"PASSWORD\"] = \"CHANGEME\"' \
                >> ~/.pootle/pootle.conf \
           && pootle migrate \
           && pootle initdb \
           && echo \"from django.contrib.auth import get_user_model; \
                     user = get_user_model().objects.create_superuser('admin', \
                                                                      'admin@example.com', \
                                                                      'admin'); \" \
                | pootle shell \
           && pootle verify_user admin \
           && rm ~/.pootle/pootle.conf \
           && pootle init \
                  --db=postgresql \
           && pootle migrate \
           && pootle initdb \
           && echo \"from django.contrib.auth import get_user_model; \
                     user = get_user_model().objects.create_superuser('admin', \
                                                                      'admin@example.com', \
                                                                      'admin'); \" \
                | pootle shell \
           && pootle verify_user admin \
           && rm ~/.pootle/pootle.conf \
           && pip install -r src/pootle/requirements/dev.txt \
           && cp src/pootle/pootle/settings/90-dev-local.conf.sample ~/.pootle/pootle.conf \
           && cd src/pootle/pootle/static/js/ \
           && npm install \
           && mkdir $INSTALL_DIR/src/pootle/pootle/assets \
           && pootle webpack \
           && pootle collectstatic \
                  --noinput \
                  --clear \
                  -i node_modules \
                  -i *.jsx \
           && pootle assets build \
           && pootle migrate \
           && pootle initdb --no-rq \
           && echo \"from django.contrib.auth import get_user_model; \
                     user = get_user_model().objects.create_superuser('admin', \
                                                                      'admin@example.com', \
                                                                      'admin'); \" \
                | pootle shell \
           && pootle verify_user admin \
           && echo 'DEBUG = True' \
                >> ~/.pootle/pootle.conf \
           && sleep 300" \
    && /etc/init.d/redis-server stop \
    && /etc/init.d/mysql stop \
    && /etc/init.d/postgresql stop


CMD /etc/init.d/redis-server start \
    && /etc/init.d/postgresql start \
    && /etc/init.d/mysql start \
    && (if [ ! -z "$USERID" ]; \
          then \
            usermod -u "$USERID" pootle; \
        fi) \
    && sudo -u pootle \
         /bin/bash -c "bash --rcfile <(echo \"\
           cd $INSTALL_DIR \
           && . bin/activate \
           && cd src/pootle \
           && pip install -e . \
           && (if [ ! -e "$INSTALL_DIR/src/pootle/pootle/static/js/node_modules" ]; \
               then \
                  cd $INSTALL_DIR/src/pootle/pootle/static/js/ \
                  && npm install \
                  && cd $INSTALL_DIR; \
               fi) \
           && (pootle rqworker &) \
           && (pootle webpack --dev &) \")" \
    && /etc/init.d/redis-server stop \
    && /etc/init.d/postgresql stop \
    && /etc/init.d/mysql stop


EXPOSE 8000
