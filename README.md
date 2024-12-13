# QWC2 Docker Setup

Following this guide you will be able to get a running instance of Qgis Web Client 2 with Giswater. This instance is composed of:

- `qwc2-docker` (this repository): Dockerized containers for QWC Services, Giswater Service, QGIS Server, Nginx and others.
- `qwc2-giswater-app`: QWC2 ReactJS and OpenLayers application customized with Giswater and other features.

## Prerequisites

- Docker
- Nginx
- Nodejs
- npm
- yarn

Installation process may differ in each system, skip installation steps if the software is already installed.

## Install docker (Debian):

- Update the apt package index and install packages to allow apt to use a repository over HTTPS:
    ~~~~
    sudo apt-get update
    sudo apt-get install ca-certificates curl gnupg
    ~~~~
- Add Docker’s official GPG key:
    ~~~~
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg
    ~~~~

- Use the following command to set up the repository:

    ~~~~
    echo \
    "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
    "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ~~~~

- Update the apt package index:
    ~~~~
    sudo apt-get update
    ~~~~

- Install Docker Engine, containerd, and Docker Compose.
    ~~~~
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ~~~~


- Verify that the Docker Engine installation is successful by running the hello-world image:
    ~~~~
    sudo docker run hello-world
    ~~~~

## Install docker (Ubuntu Server):

- Update the apt package index and install packages to allow apt to use a repository over HTTPS:
    ~~~~
    sudo apt-get update
    sudo apt install apt-transport-https ca-certificates curl software-properties-common
    ~~~~
- Add Docker’s official GPG key:
    ~~~~
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ~~~~

- Add Docker's repository to APT:

    ~~~~
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    ~~~~

- Update the apt package index:
    ~~~~
    sudo apt-get update
    ~~~~

- Install Docker Engine, containerd, and Docker Compose.
    ~~~~
    sudo apt install docker-ce docker-ce-cli containerd.io
    ~~~~

 - Verify that the Docker Engine installation is successful by running the hello-world image:
    ~~~~
    sudo docker run hello-world
    ~~~~
## Clone repository and further steps
Clone qwc2-docker repository



- Clone the new branch in bgeoadmin's home directory:

    ~~~~
    git clone --recursive https://github.com/bgeo-gis/qwc2-docker.git
    ~~~~

- Give permissions to the volumes directory:

    ~~~~
    sudo chown -R 33:33 qwc2-docker/qwc-docker/volumes
    ~~~~

- Generate a new `JWT_SECRET_KEY`
    ~~~~
    python3 -c 'import secrets; print("JWT_SECRET_KEY=\"%s\"" % secrets.token_hex(48))' > qwc-docker/.env
    ~~~~
- Make sure that `qwc-qgis-server/entrypoint.sh` is executable:

    ~~~~
    sudo chmod +x qwc-qgis-server/entrypoint.sh
    ~~~~

- Change the Host and Port in the `nginx.conf` file for `/admin`, `/auth` and `/`
    ~~~~
        location ~ ^/(?<t>bgeo)/auth {
                # set tenant header
                proxy_set_header Tenant $t;
                proxy_set_header Host localhost:8088; <---
                # remove tenant prefix
                rewrite ^/[^/]+(.+) $1 break;
                # forward to Auth Service
                proxy_pass http://qwc-auth-service:9090;
            }
    ~~~~

- Make changes in docker-compose.yml if necessary

    - Set the /data dir of qwc-qgis-server and other volumes that need it:
        ~~~~
        volumes:
            - ./volumes/qgs-resources:/data:ro
        ~~~~

    - And any other changes i.e. changing the map-viewer qwc2 build, etc.


## Docker compose
With Docker compose we can run all the application's services configured in the YAML configuration file. Now we shoud be able to start, stop and rebuild services.


- Start all services (deatached) in order to see the log output of running services

    ~~~~
    sudo docker compose up -d --build
    ~~~~


## Nginx

- Install nginx
    ~~~~
    apt install nginx
    ~~~~
- Create the file `<host>.conf` in `/etc/nginx/sites-available` to proxy pass to Docker port. In production change this configuration by adding HTTPS.
    ~~~~
    server {

        listen 80;
        server_name <host>;

        location / {
            proxy_pass http://localhost:8088/;
        }
    }
    ~~~~

- Create symbolic link between `/nginx/sites-available` and `/nginx/sites-enabled`
    ~~~~
     ln -s /etc/nginx/sites-available/xxx.conf /etc/nginx/sites-enabled/
    ~~~~
- Restart nginx
    ~~~~
    systemctl restart nginx
    ~~~~

## Client: qwc2-giswater-app

- Clone qwc2-giswater-app:

    ~~~~
    git clone --recursive --branch dev https://github.com/bgeo-gis/qwc2-giswater-app.git
    ~~~~


- Install node.js and npm:
    ~~~~
    sudo apt update
    ~~~~
    ~~~~
    sudo apt install nodejs npm
    ~~~~
- Import Yarn GPG key
    ~~~~
    # GPG KEY
    curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
    
    # Repository
    echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
    ~~~~

- Update and install yarn
    ~~~~
    sudo apt update
    ~~~~
    ~~~~
    sudo apt install yarn
    ~~~~
- To check node.js, npm and yarn versions:
    ~~~~
    node -v
    ~~~~
    ~~~~
    npm -v
    ~~~~
    ~~~~
    yarn --version
    ~~~~

- Installation:

  - Navigate to `qwc2-giswater-app` and run the following command to install `qwc2` and `qwc2-giswater` dependencies
    ~~~~
    yarn install
    ~~~~
- Change ``deploy_prod.sh`` and ``deploy_dev.sh`` to point to the correct `qwc2-docker` path if necessary.
    ~~~~
    yarn run prod
    rm -rf /qwc2-docker/qwc-docker/volumes/qwc2/*
    cp -R /qwc2-giswater-app/prod/* /qwc2-docker/qwc-docker/volumes/qwc2/
    chown -R 33:33 /qwc2-docker/qwc-docker/volumes/qwc2/assets
    ~~~~

- Deploy to test
    ~~~~
    sudo bash deploy_prod.sh
    ~~~~
## Other docker commands
- Start all services

    ~~~~
    sudo docker compose up
    ~~~~

- Build specific service
    ~~~~
    sudo docker compose up <service_name>
    ~~~~

- Stop services

   ~~~~
    sudo docker compose down
    ~~~~

# Add tenant

QWC2 offers multi-tenancy, in this repository there is example configuration for the tenant 'bgeo'. To add a new tenant follow these steps:


## First steps:

- The first step is to create a new `directory` for the new tenant in `qwc-docker/volumes/config-in`:
    ~~~~
    sudo mkdir <tenant>
    ~~~~
- Copy the `config.json`, `index.html` and `tenantConfig.json` from tenant `bgeo` to the new tenant config-in directory.


- Change the URL from `/bgeo` to `/<tenant>` in the `config.json` file

- You can change the `title` and the `favicon` in the `index.html` file



## Config DB
- Add a new entry in te `pg_service.conf` file for the QWC2 config database:
    ~~~~
    [qwc_configdb_<tenant>]
    host=qwc-config-db-<tenant>
    port=5432
    dbname=qwc_services
    user=qwc_admin
    password=qwc_admin
    sslmode=disable
    ~~~~

    We need a separate config database for each tenant, as this database will be used to store all qwc2 configuration regarding users, permissions, permalinks, etc.
    Note that to acces the ``admin backoffice`` you need to use the default user and password ``admin:admin``. After login you will be asked to change the password.

## Geo DB

- Add a new entry in the `pg_service.conf` file for the geo db. This is the database were our data is stored and it's necessary to add the geo db credentials for the Giswaer services to work.

    ~~~~
    [qwc_geodb_<tenant>]
    host=<host>
    port=<port>
    dbname=<dbname>
    user=<user>
    password=<password>
    ~~~~


## Docker compose

- Add the new tenant's config-db's definition in the `docker-compose.yml` file.

    ~~~~
   qwc-config-db-<tenant>:
     image: sourcepole/qwc-base-db:15
     environment:
       POSTGRES_PASSWORD: '' # TODO: Set your postgres password here!
     volumes:
      - ./volumes/db/<tenant>:/var/lib/postgresql/docker
     #ports:
     #- "127.0.0.1:5439:5432"
     healthcheck:
       test: ["CMD-SHELL", "pg_isready -U postgres"]
       interval: 10s

  qwc-config-db-migrate-<tenant>:
     image: sourcepole/qwc-base-db-migrate:v2024.07.24
     environment:
       PGSERVICE: 'qwc_configdb_<tenant>'
     volumes:
      - ./pg_service.conf:/tmp/pg_service.conf:ro
    ~~~~

Note: Every tenant must have a separate config-db and config-db-migrate

## Finish configuration

- Change the references from `bgeo` to `<tenant>` in the `tenantConfig.json` file, including the `config_db_url` references. 

    ~~~~
    ...

    "config_db_url": "postgresql:///?service=qwc_configdb_<tenant>",

    ...
    ~~~~

- Make sure de `qwc-docker/volumes` has permissions

    ~~~~
    sudo chown -R 33:33 qwc-docker/volumes
    ~~~~

- Add the new `tenant` in the docker `nginx.conf`
    ~~~~
        location ~ ^/(?<t>bgeo|<tenant>)/auth {
        # set tenant header
        proxy_set_header Tenant $t;
        # for missing trailing slash (use actual server host and port instead of localhost:8088)
        proxy_set_header Host 10.60.2.18;
        # remove tenant prefix
        rewrite ^/[^/]+(.+) $1 break;
        # forward to Auth Service
        proxy_pass http://qwc-auth-service:9090;
    }
    ~~~~

    Be sure to add this new tenant everywhere needed.

- Delete the themes from the tenant `bgeo` from `tenatnConfig.json` or the admin backoffice.

Create a new theme or themes and finish the configuration to start working with them. In QWC2 a 'theme' is related to a QGIS project:

- Create a new folder for the tenant's projects in `qwc-docker/volumes/qgis-resources`
- Add the desired `.qgs` projects to this newly created folder.
- Add permisions for www-data again:
    ~~~~
    sudo chown -R 33:33 qwc-docker/volumes/
    ~~~~
- Rebuild docker in order to apply the changes

  ~~~~
  docker compose up -d --build
  ~~~~
- Genereate configurations for the new tenant without using the admin backoffice

    ~~~~
    curl -X POST "http://localhost:5010/generate_configs?tenant=<tenant>"
    ~~~~

- Create the `giswaterConfig.json` file in `qwc-docker/volumes/config/<tenant>` and add the configuration for the new themes, for example:

    ~~~~
        {
        "service": "giswater",
        "config": {
            "db_url_read": "postgresql:///?service=qwc_geodb_<tenant>",
            "db_url_write: "postgresql:///?service=qwc_geodb_<tenant>",
            "images_path": "/images/<tenant>/img/",
            "images_url": "https://<url>/<tenat>/images/",
            "profile_svg_path": "/images/<tenant>/profile/",
            "themes": {
                "<Theme name>": {
                    "schema": "<schema name>",
                    "layers": {
                        "v_edit_arc": "v_edit_arc",
                        "v_edit_node": "v_edit_node",
                        "v_edit_link": "v_edit_link",
                        "v_edit_connec": "v_edit_connec",
                        "v_edit_gully": "v_edit_gully"
                    }
                }
            }
        }
    }
    ~~~~

- Edit the `config.json` file and change the plugin configuration and visibility for each theme.

- Generate configurations (from Admin Backoffice or using confi generator via terminal)

- Add the new ``tenant`` in the `nginx.conf` file for all the necessary ``locations``.
    ~~~~
        location ~ ^/(?<t>bgeo|<new_tenant>)/auth {
                # set tenant header
                proxy_set_header Tenant $t;
                proxy_set_header Host localhost:8088; <---
                # remove tenant prefix
                rewrite ^/[^/]+(.+) $1 break;
                # forward to Auth Service
                proxy_pass http://qwc-auth-service:9090;
            }
    ~~~~

- Rebuild docker in order to apply the changes

  ~~~~
  docker compose up -d --build
  ~~~~

- Change admin password.

    You should be able to acces the admin backoffice at `https://<host>/<new_tenant>/admin`. The default user and password are `admin:admin`. You will be asked to change the default password when logging in for the first time. Please change the password for a secure one.
