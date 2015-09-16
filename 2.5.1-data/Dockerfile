# translate/pootle:2.5.1
#
# VERSION	0.0.1

FROM translate/pootle:2.5.1

MAINTAINER Ryan Northey <ryan@synca.io>

RUN apt-get update \
    && apt-get install -y byobu emacs23-nox sqlite3 procps git curl

RUN sudo -u pootle bash -c " \
      cd ~/dev/env \
      && . bin/activate \

CMD byobu
