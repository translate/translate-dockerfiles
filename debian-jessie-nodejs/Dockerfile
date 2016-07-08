# translate/pootle:debian-jessie-nodejs
#
# VERSION       0.0.1

FROM translate/pootle:debian-jessie

MAINTAINER Ryan Northey <ryan@synca.io>

RUN (curl -sL https://deb.nodesource.com/setup_0.12 | sudo bash -) \
    && apt-get update \
    && sudo apt-get install -y nodejs \
    && update-alternatives --install \
         /usr/bin/node node /usr/bin/nodejs 99 \
    && (curl https://www.npmjs.com/install.sh | sh) \
    && apt-get clean
