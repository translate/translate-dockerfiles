# translate/pootle:2.7.1-mysql-migration
#
# VERSION       0.0.1

FROM translate/pootle:2.5.1-mysql

MAINTAINER Ryan Northey <ryan@synca.io>

ADD data/ /

RUN chown -R pootle:pootle /home/pootle/ \
    && echo "deb http://httpredir.debian.org/debian wheezy-backports main" \
         >> /etc/apt/sources.list \
    && apt-get update \
    && DEBIAN_FRONTEND=noninteractive \
         apt-get install \
             -t wheezy-backports \
             -y \
           redis-server \
    && /etc/init.d/mysql start \
    && /etc/init.d/redis-server start \
    && sudo -u pootle bash -c " \
         cd ~/dev/env \
	 && . bin/activate \
         && pip install --upgrade \"Pootle>=2.6,<2.7\" \
         && pootle setup \
         && pip install --upgrade /home/pootle/Pootle-2.7.1.tar.bz2 \
         && sed -i \
              s/django.db.backends/transaction_hooks.backends/g ~/.pootle/pootle.conf \
	 && echo \"DATABASES['default']['ATOMIC_REQUESTS'] = True\" >> ~/.pootle/pootle.conf \
         && echo 'DEBUG=True' >> ~/.pootle/pootle.conf \
         && pootle migrate accounts 0002 --fake \
         && pootle migrate pootle_translationproject 0002 --fake \
         && pootle migrate \
	 && pootle update_user_email admin admin@example.com \
	 && pootle verify_user admin \
	 && pootle calculate_checks --no-rq \
	 && pootle refresh_stats --no-rq \
	 && sleep 300" \
    && /etc/init.d/mysql stop \
    && /etc/init.d/redis-server stop


CMD /etc/init.d/mysql start \
    && /etc/init.d/redis-server start \
    && sudo -u pootle bash -c "\
         cd ~/dev/env \
         && . bin/activate \
         && (pootle rqworker &) \
         && pootle runserver --insecure 0.0.0.0:8000"
