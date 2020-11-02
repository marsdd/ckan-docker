# ckan-docker

Ckan docker is modified docker of ckan created as a separate project to 
adapt to our Mars' Avin project deployments on aws.  It is the purpose build 
python 3 first rebuild of the ckan docker.

# Quick start

## Install docker tools 

* https://docs.docker.com/get-docker/
* https://docs.docker.com/compose/install/
* Using environment variables via https://docs.docker.com/compose/env-file
/ method.  Check out .env.template file for example on how to fill this.

##  Clone the repository

Clone: 
    
    git clone git@github.com:marsdd/ckan-docker.git
    
OR: 

    git clone https://github.com/marsdd/ckan-docker.git
    

    
## Start the app

Start app:

    cd /path/to/ckan-docker
    docker-compose up
    
    
## Start only the non app resources

Start database, solr and redis only::

    cd /path/to/ckan-docker
    docker-compose up db solr redis
    
## Start the background workers

Start worker::

    cd /path/to/ckan-docker
    docker-compose up ckan-bg
    
That's it, that should do it.  You are up and running locally.  It may
require you to restart couple of times as data is built, but otherwise that's 
all that's needed.

