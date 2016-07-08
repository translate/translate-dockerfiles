# translate/pootle:2.8-stage
#
# VERSION       0.0.1

FROM translate/pootle:debian-jessie

MAINTAINER Ryan Northey <ryan@synca.io>

ENV INSTALL_DIR=~/pootle_env

ARG POOTLE_PKG="Pootle"

RUN /etc/init.d/redis-server start \
    && sudo -u pootle \
         bash -c "\
           mkdir -p $INSTALL_DIR \
           && cd $INSTALL_DIR \
           && virtualenv . \
           && . bin/activate \
           && pip install $POOTLE_PKG \
           && pootle init \
           && pootle migrate \
           && pootle initdb --no-rq --no-projects \
           && pootle revision --restore --no-rq \
           && echo \"from django.contrib.auth import get_user_model; \
                     user = get_user_model().objects.create_superuser('admin', \
                                                                      'admin@example.com', \
                                                                      'admin'); \" \
                | pootle shell \
           && pootle verify_user admin \
           && echo 'ALLOWED_HOSTS = [\"*\"]' \
                >> ~/.pootle/pootle.conf "\
    && redis-cli save \
    && /etc/init.d/redis-server stop

CMD /etc/init.d/redis-server start \
    && sudo -u pootle \
         bash -c "\
           cd $INSTALL_DIR \
           && . bin/activate \
           && pip install -r extra_requirements.txt \
           && pootle migrate \
           && (pootle rqworker &) \
           && mkdir /tmp/pootle_fs \
           && pootle runserver --insecure 0.0.0.0:8000" \
    && /etc/init.d/redis-server stop

EXPOSE 8000
