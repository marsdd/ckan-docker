# See CKAN docs on installation from Docker Compose on usage
FROM python:3.8.5-slim-buster AS builder
MAINTAINER MaRS Discovery District

RUN apt-get -q -y update \
    && DEBIAN_FRONTEND=noninteractive apt-get -q -y upgrade

# install debug tools in case it's needed
# RUN apt-get -y install wget curl vim

#install required packages for build and run
RUN apt-get -q -y update && \
    apt-get -y install \
    git-core \
    redis-tools \
    libpq-dev \
    build-essential \
    postgresql-client \
    curl

## Install required system packages
#RUN apt-get -q -y update \
#    && DEBIAN_FRONTEND=noninteractive apt-get -q -y upgrade \
#    && apt-get -q -y install \
#        python-dev \
#        python-pip \
#        python-virtualenv \
#        python-wheel \
#        python3-dev \
#        python3-pip \
#        python3-virtualenv \
#        python3-wheel \
#        libpq-dev \
#        libxml2-dev \
#        libxslt-dev \
#        libgeos-dev \
#        libssl-dev \
#        libffi-dev \
#        postgresql-client \
#        build-essential \
#        git-core \
#        vim \
#        wget \
#        curl \
#    && apt-get -q clean \
#    && rm -rf /var/lib/apt/lists/*
#
#RUN apt-get update && apt-get install -y redis-tools

# Define environment variables
ENV CKAN_HOME=/usr/lib/ckan
ENV CKAN_VENV=$CKAN_HOME/venv
ENV CKAN_CONFIG=$CKAN_HOME/config
ENV CKAN_STORAGE_PATH=/var/lib/ckan
ENV CKAN_INI=$CKAN_CONFIG/production.ini

# Build-time variables specified by docker-compose.yml / .env
ARG CKAN_SITE_URL
ARG MARS_PLUGIN_VERSION
ARG MARS_MULTILANG_PLUGIN_VERSION
ARG MARS_CKAN_VERSION
ARG GAID

# Create ckan user
RUN useradd -r -u 900 -m -c "ckan account" -d $CKAN_HOME -s /bin/false ckan

### Setup virtual environment for CKAN
RUN mkdir -p $CKAN_CONFIG $CKAN_STORAGE_PATH

RUN pip install virtualenv

RUN mkdir -p $CKAN_CONFIG && \
    virtualenv $CKAN_VENV &&\
    ln -s $CKAN_VENV/bin/pip /usr/local/bin/ckan-pip

ADD https://raw.githubusercontent.com/marsdd/ckan/$MARS_CKAN_VERSION/requirement-setuptools.txt \
    https://raw.githubusercontent.com/marsdd/ckan/$MARS_CKAN_VERSION/requirements.txt \
    https://raw.githubusercontent.com/marsdd/ckan/$MARS_CKAN_VERSION/wsgi.py \
    $CKAN_HOME/

ADD https://raw.githubusercontent.com/marsdd/ckan/$MARS_CKAN_VERSION/ckan/config/who.ini \
    $CKAN_CONFIG/

# Setup core ckan
RUN ckan-pip install -r $CKAN_HOME/requirement-setuptools.txt

RUN ckan-pip install -r $CKAN_HOME/requirements.txt

RUN ckan-pip install https://github.com/marsdd/ckan/archive/$MARS_CKAN_VERSION.zip
#USER ckan
#CMD ["python", "-c", "print(sys.path)"]

RUN ckan-pip install -e git+https://github.com/ckan/ckanext-googleanalytics.git#egg=ckanext-googleanalytics && \
  ckan-pip install -r $CKAN_VENV/src/ckanext-googleanalytics/requirements.txt

RUN ckan-pip install uwsgi
#
## Setup CKAN modules
ADD ./ckan-entrypoint.sh $CKAN_HOME/ckan-entrypoint.sh
RUN ckan-pip install https://github.com/marsdd/ckanext-marsavin/archive/$MARS_PLUGIN_VERSION.zip && \
  ckan-pip install https://github.com/marsdd/ckanext-multilang/archive/$MARS_MULTILANG_PLUGIN_VERSION.zip && \
  ln -s $CKAN_VENV/bin/ckan /usr/local/bin/ckan && \
  ln -s $CKAN_VENV/bin/uwsgi /usr/local/bin/uwsgi && \
  chown -R ckan:ckan $CKAN_HOME $CKAN_VENV $CKAN_CONFIG $CKAN_STORAGE_PATH && \
  chmod +x $CKAN_HOME/ckan-entrypoint.sh

RUN ckan generate config "$CKAN_INI"

## m.m. - replace Google Analytics ID
RUN sed -i "s/GAID/$GAID/g" "$CKAN_INI"

ENTRYPOINT ["/usr/lib/ckan/ckan-entrypoint.sh"]

USER ckan
EXPOSE 5000

#CMD uwsgi
##CMD ["ckan","run", "-H", "0.0.0.0", "-p]
CMD ["uwsgi", "--http", ":5000", "--wsgi-file", "/usr/lib/ckan/wsgi.py", "--master", "--enable-threads", "--need-app"]

