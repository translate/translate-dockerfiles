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
           git \
           locales \
           libxml2-dev \
           libxslt-dev \
           nodejs \
           npm \
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
    && update-alternatives --install \
         /usr/bin/node node /usr/bin/nodejs 99 \
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
    && /etc/init.d/redis-server start \
    && sudo -u pootle \
         bash -c "\
           mkdir -p ~/dev/env \
           && cd ~/dev/env \
           && virtualenv . \
           && . bin/activate \
           && pip install \
                  --upgrade \
                  -e git://github.com/translate/pootle#egg=pootle \
                psycopg2 \
	   && pootle init \
	          --db=postgresql \
		  --db-password=CHANGEME \
           && pootle migrate --noinput \
           && pootle initdb \
           && cd src/pootle/pootle/static/js/ \
           && npm install \
           && npm update \
           && mkdir ~/dev/env/src/pootle/pootle/assets \
           && pootle webpack \
           && pootle collectstatic \
                  --noinput \
                  --clear \
                  -i node_modules \
                  -i *.jsx \
           && pootle assets build \
           && echo \"from django.contrib.auth import get_user_model; \
                     from allauth.account.models import EmailAddress; \
                     from django.core import management; \
                     from django.conf import settings; \
                     user = get_user_model().objects.create_superuser('admin', 'admin@example.com', 'admin'); \
                     EmailAddress.objects.create(user=user, email='admin@example.com', primary=True, verified=True); \
                     [q.__setitem__('ASYNC', False) \
                      for q in settings.RQ_QUEUES.itervalues()]; \
                     management.call_command('refresh_stats_rq'); \" \
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
