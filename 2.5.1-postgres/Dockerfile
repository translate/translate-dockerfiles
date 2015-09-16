# translate/pootle:2.5.1-postgres
#
# VERSION       0.0.1

FROM debian:wheezy

MAINTAINER Ryan Northey <ryan@synca.io>


RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive \
         apt-get install -y \
           python-pip \
           libxml2-dev \
           libxslt-dev \
           python-dev \
           zlib1g-dev \
           build-essential \
           sudo \
           curl \
	   locales \
           postgresql-9.1 \
	   postgresql-server-dev-9.1 \
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
	      pootledb"

ADD data/ /

RUN chown -R pootle:pootle /home/pootle/ \
    && sudo -u pootle bash -c "\
         mkdir ~/dev/env -p \
         && cd ~/dev/env \
         && virtualenv . \
         && . bin/activate \
         && pip install pootle==2.5.1.3 \
                        psycopg2 "
                      
RUN chown -R pootle:pootle /home/pootle/ \
    && /etc/init.d/postgresql start \
    && /etc/init.d/postgresql restart \
    && sudo -u pootle bash -c "\
         pootle setup \
         && pootle collectstatic --noinput \
         && pootle assets build \
         && echo \"from django.contrib.auth.models import User; \
                   user = User.objects.create_superuser('pootle', \
                                                        'pootle@example.com', \
                                                        'pootle'); \" \
              | pootle shell \
         && (pootle runserver --insecure 0.0.0.0:8000 &) \
         && sleep 2 \
         && curl \
                -s \
                -L \
                -o site.html \
              http://127.0.0.1:8000/ \
         && rm site.html"

CMD sudo -u pootle \
      bash -c "\
        cd ~/dev/env \
        && . bin/activate \
        && pootle runserver --insecure 0.0.0.0:8000 "

EXPOSE 8000
