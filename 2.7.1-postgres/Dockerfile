# translate/pootle:2.7.1-postgres
#
# VERSION       0.0.1

FROM debian:jessie

MAINTAINER Ryan Northey <ryan@synca.io>

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive \
         apt-get install -y \
           build-essential \
           coreutils \
           locales \
           libxml2-dev \
           libxslt-dev \
           postgresql-9.4 \
           postgresql-server-dev-9.4 \
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
    && /etc/init.d/postgresql start \
    && sudo -u postgres bash -c "\
         psql -c \"CREATE USER pootle WITH PASSWORD 'CHANGEME';\" \
         && createdb \
                --encoding='utf-8' \
                --locale=en_US.utf8 \
                --template=template0 \
                --owner=pootle \
              pootledb" \
    && /etc/init.d/postgresql stop

ADD data/ /

RUN chown -R pootle:pootle /home/pootle \
    && /etc/init.d/postgresql start \
    && /etc/init.d/redis-server start \
    && sudo -u pootle \
         bash -c "\
           mkdir -p ~/dev/env \
           && cd ~/dev/env \
           && virtualenv . \
           && . bin/activate \
           && pip install psycopg2 \
	   && pip install /home/pootle/Pootle-2.7.1.tar.bz2 \
	   && pootle init \
	          --db=postgresql \
           && pootle migrate \
           && pootle initdb \
           && echo \"from django.contrib.auth import get_user_model; \
                     user = get_user_model().objects.create_superuser('admin', 'admin@example.com', 'admin'); \" \
                | pootle shell \
           && sleep 300" \
    && /etc/init.d/postgresql stop \
    && /etc/init.d/redis-server stop


CMD /etc/init.d/postgresql start \
    && sleep 2 \    
    && /etc/init.d/redis-server start \
    && sudo -u pootle \
         bash -c "\
           cd ~/dev/env \
           && . bin/activate \
           && (pootle rqworker &) \
           && pootle runserver --insecure 0.0.0.0:8000"

EXPOSE 8000
