# translate/pootle:2.8-build
#
# VERSION       0.0.1

FROM translate/pootle:debian-jessie-nodejs

MAINTAINER Ryan Northey <ryan@synca.io>

ENV INSTALL_DIR=~/pootle_env

ARG POOTLE_BRANCH=http://github.com/translate/pootle

COPY data/pootle_build.conf /home/pootle/.pootle/pootle.conf

RUN /etc/init.d/redis-server start \
    && sudo -u pootle \
      bash -c "\
        mkdir -p $INSTALL_DIR \
        && cd $INSTALL_DIR \
        && virtualenv . \
        && . bin/activate \
        && mkdir src \
        && pip install -e \"git+$POOTLE_BRANCH#egg=Pootle\" \
        && pip install -r ~/pootle_env/src/pootle/requirements/build.txt \
        && export POOTLE_SETTINGS=~/.pootle/pootle.conf \
        && export DJANGO_SETTINGS_MODULE=pootle.settings \
        && cd ~/pootle_env/src/pootle \
        && ./setup.py build_checks_templates \
        && cd ~/pootle_env/src/pootle/pootle/static/js \
        && npm install lodash \
        && cd ~/pootle_env/src/pootle \
        && mkdir assets \
        && make assets \
        && make build" \
   && /etc/init.d/redis-server stop

CMD /etc/init.d/redis-server start \
    && sudo -u pootle \
      /bin/bash -c "bash --rcfile <(echo \"\
        cd $INSTALL_DIR \
        && . bin/activate \
        && export POOTLE_SETTINGS=~/.pootle/pootle.conf \
        && export DJANGO_SETTINGS_MODULE=pootle.settings \
        && cd src/pootle  \")"

EXPOSE 8000
