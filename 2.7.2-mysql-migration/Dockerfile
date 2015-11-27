# translate/pootle:2.7.2-mysql-migration
#
# VERSION       0.0.1

FROM translate/pootle:2.7.1-mysql-migration

MAINTAINER Ryan Northey <ryan@synca.io>

ENV POOTLE_PKG='Pootle==2.7.2'

RUN /etc/init.d/mysql start \
    && /etc/init.d/redis-server start \
    && sudo -u pootle \
         bash -c " \
           cd $INSTALL_DIR \
           && . bin/activate \
           && pip install \
  	          --upgrade \
		$POOTLE_PKG \
           && sleep 300" \
    && /etc/init.d/mysql stop \
    && /etc/init.d/redis-server stop


CMD /etc/init.d/mysql start \
    && /etc/init.d/redis-server start \
    && sudo -u pootle \
         bash -c "\
           cd $INSTALL_DIR \
           && . bin/activate \
           && (pootle rqworker &) \
           && pootle runserver --insecure 0.0.0.0:8000"
