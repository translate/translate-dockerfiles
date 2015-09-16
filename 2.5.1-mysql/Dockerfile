# translate/pootle:2.5.1-postgres
#
# VERSION       0.0.1

FROM debian:wheezy

MAINTAINER Ryan Northey <ryan@synca.io>


RUN apt-get update -qq \
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
	   locales \
           mysql-server \
	   libmysqlclient-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
    && echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
    && locale-gen \
    && update-locale LANG=en_US.UTF-8 \
    && pip install virtualenv \
    && groupadd -r pootle \
    && useradd \
           -m \
           -d /home/pootle \
           -k /etc/skel \
           -s /bin/bash \
           -g pootle \
         pootle

ADD data/ /

RUN chown -R pootle:pootle /home/pootle/ \
    && /etc/init.d/mysql start \
    && mysql -e "\
         CREATE DATABASE pootledb \
	   CHARACTER SET utf8 \
	   DEFAULT COLLATE utf8_general_ci; \
         GRANT ALL PRIVILEGES ON pootledb.* \
	   TO pootle@localhost \
	   IDENTIFIED BY 'CHANGEME'; \
	 FLUSH PRIVILEGES;" \	 
    && sudo -u pootle bash -c "\
         mkdir ~/dev/env -p \
         && cd ~/dev/env \
         && virtualenv . \
         && . bin/activate \
         && pip install pootle==2.5.1.3 \
                        MySQL-python \
         && pootle setup \
         && pootle collectstatic --noinput \
         && pootle assets build \
         && (pootle runserver --insecure 0.0.0.0:8000 &) \
         && sleep 2 \
         && curl \
                -s \
                -L \
                -o site.html \
              http://127.0.0.1:8000/ \
         && rm site.html \
	 && sleep 300" \
       && /etc/init.d/mysql stop

CMD /etc/init.d/mysql start \
    && sudo -u pootle \
         bash -c "\
           cd ~/dev/env \
           && . bin/activate \
           && pootle runserver --insecure 0.0.0.0:8000 "

EXPOSE 8000
