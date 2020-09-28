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

# Define environment variables
ENV CKAN_HOME=/usr/lib/ckan
ENV CKAN_VENV=$CKAN_HOME/venv
ENV CKAN_CONFIG=$CKAN_HOME/config
ENV CKAN_INI=$CKAN_CONFIG/production.ini

# Build-time variables specified by docker-compose.yml / .env
ARG CKAN_SITE_URL
ARG MARS_PLUGIN_VERSION
ARG MARS_MULTILANG_PLUGIN_VERSION
ARG MARS_CKAN_VERSION
ARG GAID

### Setup virtual environment for CKAN
RUN mkdir -p $CKAN_CONFIG

RUN pip install virtualenv

RUN mkdir -p $CKAN_CONFIG && \
    virtualenv $CKAN_VENV &&\
    ln -s $CKAN_VENV/bin/pip /usr/local/bin/ckan-pip

ADD https://raw.githubusercontent.com/marsdd/ckan/$MARS_CKAN_VERSION/requirement-setuptools.txt \
    https://raw.githubusercontent.com/marsdd/ckan/$MARS_CKAN_VERSION/requirements.txt \
    https://raw.githubusercontent.com/marsdd/ckan/$MARS_CKAN_VERSION/dev-requirements.txt \
    $CKAN_HOME/

# Setup core ckan
RUN ckan-pip install -r $CKAN_HOME/requirement-setuptools.txt

RUN ckan-pip install -r $CKAN_HOME/requirements.txt

RUN ckan-pip install -r $CKAN_HOME/dev-requirements.txt

RUN ckan-pip install https://github.com/marsdd/ckan/archive/$MARS_CKAN_VERSION.zip
#USER ckan
#CMD ["python", "-c", "print(sys.path)"]

RUN ckan-pip install -e git+https://github.com/ckan/ckanext-googleanalytics.git#egg=ckanext-googleanalytics && \
  ckan-pip install -r $CKAN_VENV/src/ckanext-googleanalytics/requirements.txt

#
## Setup CKAN modules
RUN ckan-pip install https://github.com/marsdd/ckanext-marsavin/archive/$MARS_PLUGIN_VERSION.zip && \
  ckan-pip install https://github.com/marsdd/ckanext-multilang/archive/$MARS_MULTILANG_PLUGIN_VERSION.zip

# Multistage build process, this is the runner container
FROM python:3.8.5-slim-buster as RUNNER

# Define environment variables
ENV CKAN_HOME=/usr/lib/ckan
ENV CKAN_VENV=$CKAN_HOME/venv
ENV CKAN_CONFIG=$CKAN_HOME/config
ENV CKAN_INI=$CKAN_CONFIG/production.ini
ENV PGPASSWORD=$POSTGRES_PASSWORD

# Build-time variables specified by docker-compose.yml / .env
ARG CKAN_SITE_URL
ARG MARS_CKAN_VERSION
ARG GAID

# Create ckan user
RUN useradd -r -u 900 -m -c "ckan account" -d $CKAN_HOME -s /bin/false ckan

#install required packages for build and run
RUN apt-get -q -y update \
    && DEBIAN_FRONTEND=noninteractive apt-get -q -y upgrade && \
    apt-get -y install postgresql-client curl

RUN mkdir -p $CKAN_CONFIG

COPY --from=builder $CKAN_VENV $CKAN_VENV

ADD ./ckan-entrypoint.sh $CKAN_HOME/

ADD https://raw.githubusercontent.com/marsdd/ckan/$MARS_CKAN_VERSION/ckan/config/who.ini \
    $CKAN_CONFIG/

RUN ln -s $CKAN_VENV/bin/pip /usr/local/bin/ckan-pip && \
    ln -s $CKAN_VENV/bin/ckan /usr/local/bin/ckan && \
    ln -s $CKAN_VENV/bin/pytest /usr/local/bin/pytest && \
    chown -R ckan:ckan $CKAN_HOME $CKAN_VENV $CKAN_CONFIG

RUN chmod +x $CKAN_HOME/ckan-entrypoint.sh
RUN ckan generate config "$CKAN_INI"

## m.m. - replace Google Analytics ID
RUN sed -i "s/GAID/$GAID/g" "$CKAN_INI"

ENTRYPOINT ["/usr/lib/ckan/ckan-entrypoint.sh"]

USER ckan
EXPOSE 5000

#CMD ["ckan", "datastore", "set-permissions"]
CMD ["pytest", "--ckan-ini=/usr/lib/ckan/config/production.ini", "ckan/", "ckan_ext/"]
