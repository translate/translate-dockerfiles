# translate/pootle:2.7.1-mysql-migration
#
# VERSION       0.0.1

FROM translate/pootle:2.5.1-mysql

MAINTAINER Ryan Northey <ryan@synca.io>

ENV POOTLE_PKG='Pootle==2.7.1'

RUN echo "deb http://httpredir.debian.org/debian wheezy-backports main" \
       >> /etc/apt/sources.list \
    && apt-get update \
    && apt-get install \
           -t wheezy-backports \
           -y \
         redis-server \
    && /etc/init.d/mysql start \
    && /etc/init.d/redis-server start \
    && sudo -u pootle \
         bash -c " \
           cd $INSTALL_DIR \
           && . bin/activate \
           && pip install --upgrade \"Pootle>=2.6,<2.7\" \
           && pootle setup \
           && pip install \
  	          --upgrade \
		$POOTLE_PKG \
		\"elasticsearch<1.7\" \
           && rm ~/.pootle/pootle.conf \
           && pootle init --db=mysql \
           && echo 'DATABASES[\"default\"][\"PASSWORD\"] = \"CHANGEME\"' \
                >> ~/.pootle/pootle.conf \
           && pootle migrate accounts 0002 --fake \
           && pootle migrate pootle_translationproject 0002 --fake \
           && pootle migrate \
           && pootle update_user_email admin admin@example.com \
           && pootle verify_user admin \
           && pootle calculate_checks --no-rq \
           && pootle refresh_stats --no-rq \
           && echo 'ALLOWED_HOSTS = [\"*\"]' \
                >> ~/.pootle/pootle.conf \
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
