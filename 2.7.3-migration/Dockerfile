# translate/pootle:2.7.3-migration
#
# VERSION       0.0.1

FROM translate/pootle:2.7.2-migration

MAINTAINER Ryan Northey <ryan@synca.io>

ENV POOTLE_PKG='Pootle==2.7.3'

RUN /etc/init.d/redis-server start \
    && sudo -u pootle \
         bash -c " \
           cd $INSTALL_DIR \
           && . bin/activate \
           && pip install \
  	          --upgrade \
		$POOTLE_PKG \
           && pootle migrate" \
    && /etc/init.d/redis-server stop


CMD /etc/init.d/redis-server start \
    && sudo -u pootle \
         bash -c "\
           cd $INSTALL_DIR \
           && . bin/activate \
           && (pootle rqworker &) \
           && pootle runserver --insecure 0.0.0.0:8000"
