# translate/pootle:2.5.1
#
# VERSION	0.0.1

FROM pootle:2.5.1

MAINTAINER Ryan Northey <ryan@synca.io>

RUN apt-get update \
    && apt-get install -y byobu emacs23-nox sqlite3 procps git

ADD data/ /

RUN chown -R pootle:pootle /home/pootle/ \
    && echo "deb http://httpredir.debian.org/debian wheezy-backports main" \
         >> /etc/apt/sources.list \
    && apt-get update \
    && DEBIAN_FRONTEND=noninteactive \
       apt-get install \
           -t wheezy-backports \
	   -y \
         redis-server \
         nodejs-legacy \
       	 curl \
    && curl -L --insecure -O https://www.npmjs.org/install.sh \
    && /bin/bash install.sh \
    && rm install.sh \
    && /etc/init.d/redis-server start \
    && sudo -u pootle bash -c " \
         cd ~/dev/env \
      	 && . bin/activate \
      	 && pip install --upgrade \"Pootle>=2.6,<2.7\" \
      	 && pootle setup \
      	 && pip install \
	        --upgrade \
		-e \
	      git://github.com/translate/pootle#egg=pootle \
      	 && sed -i \
	      s/django.db.backends/transaction_hooks.backends/g ~/.pootle/pootle.conf \
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
      	 && pootle assets build "


RUN /etc/init.d/redis-server start \
    && sudo -u pootle bash -c " \
         cd ~/dev/env \
      	 && . bin/activate \
	 && echo \"from django.core import management; \
	    	   from django.conf import settings; \
		   [q.__setitem__('ASYNC', False) \
                    for q in settings.RQ_QUEUES.itervalues()]; \
	           management.call_command('refresh_stats_rq'); \" \
	      | pootle shell "


CMD /etc/init.d/redis-server start \
    && byobu
