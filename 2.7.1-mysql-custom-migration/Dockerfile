# translate/pootle:2.7.1-mysql-migration
#
# VERSION       0.0.1

FROM translate/pootle:2.5.1-mysql-custom

MAINTAINER Ryan Northey <ryan@synca.io>

COPY data/ /

RUN chown -R pootle:pootle /home/pootle/ \
    && echo "deb http://httpredir.debian.org/debian wheezy-backports main" \
         >> /etc/apt/sources.list \
    && apt-get update \
    && DEBIAN_FRONTEND=noninteractive \
         apt-get install \
             -t wheezy-backports \
             -y \
           redis-server \
           nodejs-legacy \
	   git \
           curl \
    && curl -L --insecure -O https://www.npmjs.org/install.sh \
    && /bin/bash install.sh \
    && rm install.sh \
    && /etc/init.d/mysql start \
    && /etc/init.d/redis-server start \
    && sudo -u pootle bash -c " \
      cd ~/dev/env \
         && . bin/activate \
         && pip install \
                --upgrade \
                -e git://github.com/phlax/pootle@2.6.1#egg=pootle \
	 && pootle setup \
         && pip install \
                --upgrade \
		--exists-action=w \
                -e git://github.com/translate/pootle#egg=pootle \
         && sed -i \
              s/django.db.backends/transaction_hooks.backends/g ~/.pootle/pootle.conf \
	 && echo \"DATABASES['default']['ATOMIC_REQUESTS'] = True\" >> ~/.pootle/pootle.conf \
         && echo 'DEBUG=True' >> ~/.pootle/pootle.conf \
 	 && echo 'yes' | pootle migrate accounts 0002 --fake \
	 && pootle migrate pootle_translationproject 0002 --fake \
	 && pootle migrate \
         && cd src/pootle/pootle/static/js/ \
         && npm install \
         && npm update \
         && mkdir -p ~/dev/env/src/pootle/pootle/assets \
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
                   user = get_user_model().objects.get(username='admin'); \
                   EmailAddress.objects.create(user=user, email='admin@example.com', primary=True, verified=True); \
                   [q.__setitem__('ASYNC', False) \
                    for q in settings.RQ_QUEUES.itervalues()]; \
                   management.call_command('refresh_stats_rq'); \" \
              | pootle shell \
	 && sleep 600" \
    && /etc/init.d/mysql stop \
    && /etc/init.d/redis-server stop


CMD /etc/init.d/mysql start \
    && /etc/init.d/redis-server start \
    && sudo -u pootle bash -c "\
         cd ~/dev/env \
         && . bin/activate \
         && (pootle rqworker &) \
         && pootle runserver --insecure 0.0.0.0:8000"
