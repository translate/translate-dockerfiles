# translate/pootle:2.5.1
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
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
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
         mkdir ~/dev/env -p \
         && cd ~/dev/env \
         && virtualenv . \
         && . bin/activate \
         && pip install pootle==2.5.1.3 \
         && pootle init \
         && pootle setup \
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
