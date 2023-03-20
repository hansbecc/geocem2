# Geocem2

GeoNode template project. Generates a django project with GeoNode support.

## Table of Contents

-  [Developer Workshop](#developer-Workshop)
-  [Create a custom project](#create-a-custom-project)
-  [Start your server using Docker](#start-your-server-using-docker)
-  [Run the instance in development mode](#run-the-instance-in-development-mode)
-  [Run the instance on a public site](#run-the-instance-on-a-public-site)
-  [Stop the Docker Images](#stop-the-docker-images)
-  [Backup and Restore from Docker Images](#backup-and-restore-the-docker-images)
-  [Recommended: Track your changes](#recommended-track-your-changes)
-  [Hints: Configuring `requirements.txt`](#hints-configuring-requirementstxt)

## Developer Workshop

Available at

  ```bash
    http://geonode.org/dev-workshop
  ```

## Create a custom project

**NOTE**: *You can call your geonode project whatever you like **except 'geonode'**. Follow the naming conventions for python packages (generally lower case with underscores (``_``). In the examples below, replace ``geocem2`` with whatever you would like to name your project.*

To setup your project follow these instructions:

1. Generate the project

    ```bash
    git clone https://github.com/GeoNode/geonode-project.git -b <your_branch>
    source /usr/share/virtualenvwrapper/virtualenvwrapper.sh
    mkvirtualenv --python=/usr/bin/python3 geocem2
    pip install Django==3.2.16

    django-admin startproject --template=./geonode-project -e py,sh,md,rst,json,yml,ini,env,sample,properties -n monitoring-cron -n Dockerfile geocem2

    cd geocem2
    ```

2. Create the .env file

    An `.env` file is requird to run the application. It can be created from the `.env.sample` either manually or with the create-envfile.py script:
    ```bash
    pyhton crate-envfile.py
    ```
    The script accepts several parameters to create the file, in detail:

    - *hostname*: e.g. master.demo.geonode.org, default localhost
    - *https*: (boolean), default value is False
    - *email*: Admin email (this is required if https is set to True since a valid email is required by Letsencrypt certbot)
    - *env_type*: `prod`, `test` or `dev`. It will set the `DEBUG` variable to `False` (`prod`, `test`) or `True` (`dev`)
    - *geonodepwd*: GeoNode admin password (required inside the .env)
    - *geoserverpwd*: Geoserver admin password (required inside the .env)
    - *pgpwd*: PostgreSQL password (required inside the .env)
    - *dbpwd*: GeoNode DB user password (required inside the .env)
    - *geodbpwd*: Geodatabase user password (required inside the .env)
    - *clientid*: Oauth2 client id (required inside the .env)
    - *clientsecret*: Oauth2 client secret (required inside the .env)
    - *secret key*: Django secret key (required inside the .env)
    - *sample_file*: absolute path to a env_sample file used to create the env_file. If not provided, the one inside the GeoNode project is used.
    - *file*: absolute path to a json file that contains all the above configuration
    
    Some important parameters configuration:
    **NOTE**: In this example we are going to publish to the public IP http://200.144.244.238

    ```bash
    vim .env
      --> replace localhost with 200.144.244.238 everywhere
    ```

    ```bash
        COMPOSE_PROJECT_NAME=geocem2
    ```
    ```bash
        DOCKER_ENV=production
    ```
    backend
    ```bash
        POSTGRES_USER=postgres
        POSTGRES_PASSWORD=postgres
        GEONODE_DATABASE=geocem2
        GEONODE_DATABASE_PASSWORD=geonode
        GEONODE_GEODATABASE=geocem2_data
        GEONODE_GEODATABASE_PASSWORD=geonode
        GEONODE_DATABASE_SCHEMA=public
        GEONODE_GEODATABASE_SCHEMA=public
        DATABASE_HOST=db
        DATABASE_PORT=5432
        DATABASE_URL=postgis://geocem2:geonode@db:5432/geocem2
        GEODATABASE_URL=postgis://geocem2_data:geonode@db:5432/geocem2_data
     ```
     ```bash
        SITEURL=http://localhost/
        ALLOWED_HOSTS="['django', '*']"
     ```     
     ```bash
        # #################
        # nginx
        # HTTPD Server
        # #################
        GEONODE_LB_HOST_IP=200.144.244.238
        GEONODE_LB_PORT=80
        PUBLIC_PORT=80
      ```
      ```bash
        # IP or domain name and port where the server can be reached on HTTPS (leave HOST empty if you want to use HTTP only)
        # port where the server can be reached on HTTPS
        HTTP_HOST=200.144.244.238
        HTTPS_HOST=200.144.244.238

        HTTP_PORT=80
        HTTPS_PORT=443
     ```
     ```bash
        # #################
        # geoserver
        # #################
        GEOSERVER_WEB_UI_LOCATION=http://200.144.244.238/geoserver/
        GEOSERVER_PUBLIC_LOCATION=http://200.144.244.238/geoserver/
        GEOSERVER_LOCATION=http://geoserver:8080/geoserver/
        GEOSERVER_ADMIN_USER=admin
        GEOSERVER_ADMIN_PASSWORD=cem238
     ```
     ```bash
        # #################
        # Security
        # #################
        # Admin Settings
        #
        # ADMIN_PASSWORD is used to overwrite the GeoNode admin password **ONLY** the first time
        # GeoNode is run. If you need to overwrite it again, you need to set the env var FORCE_REINIT,
        # otherwise the invoke updateadmin task will be skipped and the current password already stored
        # in DB will honored.

        ADMIN_USERNAME=admin
        ADMIN_PASSWORD=cem238
        ADMIN_EMAIL=None
     ```
     To allow user registration:
     ```bash
        # Users Registration
        ACCOUNT_OPEN_SIGNUP=True
     ```
     Modify the code and the templates and rebuild the Docker Containers
     ```bash
        docker-compose -f docker-compose.yml build --no-cache
     ```
     Finally, run the containers
     ```bash
        docker-compose -f docker-compose.yml up -d
     ```
     Access the site on http://200.144.244.238/


### Fully Wipe-out the Docker Images

**WARNING**: This will wipe out all the repositories created until now.

**NOTE**: The images must be stopped first

```bash
docker system prune -a
```

## Backup and Restore from Docker Images

### Run a Backup

```bash
SOURCE_URL=$SOURCE_URL TARGET_URL=$TARGET_URL ./geocem2/br/backup.sh $BKP_FOLDER_NAME
```

- BKP_FOLDER_NAME:
  Default value = backup_restore
  Shared Backup Folder name.
  The scripts assume it is located on "root" e.g.: /$BKP_FOLDER_NAME/

- SOURCE_URL:
  Source Server URL, the one generating the "backup" file.

- TARGET_URL:
  Target Server URL, the one which must be synched.

e.g.:

```bash
docker exec -it django4geocem2 sh -c 'SOURCE_URL=$SOURCE_URL TARGET_URL=$TARGET_URL ./geocem2/br/backup.sh $BKP_FOLDER_NAME'
```

### Run a Restore

```bash
SOURCE_URL=$SOURCE_URL TARGET_URL=$TARGET_URL ./geocem2/br/restore.sh $BKP_FOLDER_NAME
```

- BKP_FOLDER_NAME:
  Default value = backup_restore
  Shared Backup Folder name.
  The scripts assume it is located on "root" e.g.: /$BKP_FOLDER_NAME/

- SOURCE_URL:
  Source Server URL, the one generating the "backup" file.

- TARGET_URL:
  Target Server URL, the one which must be synched.

e.g.:

```bash
docker exec -it django4geocem2 sh -c 'SOURCE_URL=$SOURCE_URL TARGET_URL=$TARGET_URL ./geocem2/br/restore.sh $BKP_FOLDER_NAME'
```

## Recommended: Track your changes

Step 1. Install Git (for Linux, Mac or Windows).

Step 2. Init git locally and do the first commit:

```bash
git init
git add *
git commit -m "Initial Commit"
```

Step 3. Set up a free account on github or bitbucket and make a copy of the repo there.

## Hints: Configuring `requirements.txt`

You may want to configure your requirements.txt, if you are using additional or custom versions of python packages. For example

```python
Django==3.2.16
git+git://github.com/<your organization>/geonode.git@<your branch>
```

## Increasing PostgreSQL Max connections

In case you need to increase the PostgreSQL Max Connections , you can modify
the **POSTGRESQL_MAX_CONNECTIONS** variable in **.env** file as below:

```
POSTGRESQL_MAX_CONNECTIONS=200
```

In this case PostgreSQL will run accepting 200 maximum connections.

