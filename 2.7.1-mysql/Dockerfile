# translate/pootle:2.7.1-mysql
#
# VERSION       0.0.1

FROM debian:jessie

MAINTAINER Ryan Northey <ryan@synca.io>

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive \
         apt-get install -y \
           build-essential \
           coreutils \
           libmysqlclient-dev \
           libxml2-dev \
           libxslt-dev \
           locales \
           mysql-server \
           python-dev \
           python-pip \
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
    && /etc/init.d/mysql start \
    && mysql -e "\
         CREATE DATABASE pootledb \
           CHARACTER SET utf8 \
           DEFAULT COLLATE utf8_general_ci; \
         GRANT ALL PRIVILEGES ON pootledb.* \
           TO pootle@localhost \
           IDENTIFIED BY 'CHANGEME'; \
         FLUSH PRIVILEGES;" \
    && /etc/init.d/mysql stop

ADD data/ /

RUN chown -R pootle:pootle /home/pootle \
    && /etc/init.d/redis-server start \
    && /etc/init.d/mysql start \
    && sudo -u pootle \
         bash -c "\
           mkdir -p ~/dev/env \
           && cd ~/dev/env \
           && virtualenv . \
           && . bin/activate \
           && pip install MySQL-python \
	   && pip install /home/pootle/Pootle-2.7.1.tar.bz2 \
	   && pootle init \
	          --db=mysql \
	   && echo 'DATABASES[\"default\"][\"PASSWORD\"] = \"CHANGEME\"' \
	        >> ~/.pootle/pootle.conf \
           && pootle migrate \
           && pootle initdb \
           && echo \"from django.contrib.auth import get_user_model; \
                     user = get_user_model().objects.create_superuser('admin', 'admin@example.com', 'admin'); \" \
                | pootle shell \
	   && pootle verify_user admin \
	   && pootle refresh_stats --no-rq \
           && echo 'DEBUG = True' >> ~/.pootle/pootle.conf \
           && sleep 300" \
    && /etc/init.d/redis-server stop \
    && /etc/init.d/mysql stop


CMD /etc/init.d/redis-server start \
    && /etc/init.d/mysql start \
    && sudo -u pootle \
         bash -c "\
           cd ~/dev/env \
           && . bin/activate \
           && (pootle rqworker &) \
           && pootle runserver --insecure 0.0.0.0:8000"

EXPOSE 8000
