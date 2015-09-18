# translate/pootle:2.7.1
#
# VERSION       0.0.1

FROM debian:jessie

MAINTAINER Ryan Northey <ryan@synca.io>

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive \
         apt-get install -y \
           build-essential \
           coreutils \
           libxml2-dev \
           libxslt-dev \
	   locales \
           python-pip \
           python-dev \
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
         pootle 

ADD data/ /

RUN chown -R pootle:pootle
    && /etc/init.d/redis-server start \
    && sudo -u pootle \
         bash -c "\
           mkdir -p ~/dev/env \
           && cd ~/dev/env \
           && virtualenv . \
           && . bin/activate \
	   && pip install /home/pootle/Pootle-2.7.1.tar.bz2 \
           && pootle init \
           && pootle migrate --noinput \
           && pootle initdb \
           && echo \"from django.contrib.auth import get_user_model; \
                     user = get_user_model().objects.create_superuser('admin', 'admin@example.com', 'admin'); \" \
                | pootle shell \
	   && pootle verify_user admin \
	   && pootle refresh_stats_rq --no-rq \
           && echo 'DEBUG = True' >> ~/.pootle/pootle.conf \	   
           && sleep 300" \
    && /etc/init.d/redis-server stop

CMD /etc/init.d/redis-server start \
    && sudo -u pootle \
         bash -c "\
           cd ~/dev/env \
           && . bin/activate \
           && (pootle rqworker &) \
           && pootle runserver 0.0.0.0:8000"

EXPOSE 8000
