# translate/pootle:2.7.1-mysql-migration
#
# VERSION       0.0.1

FROM translate/pootle:2.7.1-mysql-migration

MAINTAINER Ryan Northey <ryan@synca.io>

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive \
         apt-get install \
             -y \
	     emacs23-nox \
	     byobu


CMD /etc/init.d/mysql start \
    && /etc/init.d/redis-server start \
    && byobu
