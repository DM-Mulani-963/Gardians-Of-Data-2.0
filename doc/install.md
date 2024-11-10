# Getting Started with DATA Rakshak (Guardians of Data)

- [Installing your development environment](#installing-your-development-environment)
- [Configuring your local instance](#configuring-your-local-instance)
- [Analyzing an application](#analyzing-an-application)

## Installing your development environment

You have different ways of setting up your development environment:

- [Docker](#docker-setup)
- [Manual](#manual-setup)
- ~~Vagrant (Deprecated)~~

### Docker setup

#### Requirements

- docker
- docker-compose

#### Prepare your settings

You can tweak your instance by changing [some settings](#configuring-your-local-instance).

Follow instructions to get AAS token [here](https://github.com/EFForg/apkeep/blob/master/USAGE-google-play.md).

Create the file  `datarakshak/settings/custom_docker.py` with the following content:

```bash
from .docker import *

GOOGLE_ACCOUNT_EMAIL = "<a valid google account>"
GOOGLE_ACCOUNT_AAS_TOKEN = "<a valid aas token>"

# Overwrite any other settings you wish to
```

#### Run

```bash
echo uid=$(id -u) > .env
docker-compose up -d
# Once DATA Rakshak started, you can check its logs
docker-compose logs -f DATA Rakshak-front
```

When everything is up (Docker logs `DATA Rakshak DB is ready.`), you may have to force a first download of the F-Droid index:

```bash
docker-compose exec DATA Rakshak-worker /entrypoint.sh refresh-fdroid-index
```

**The worker must be running** to do this.

The DATA Rakshak container automatically:

- Create the database
- Make migration
- Import trackers from the main instance
- Start the frontend of DATA Rakshak

Don't forget to rebuild your image and refresh your container if there is any change with `docker-compose up -d --build`.

#### Aliases

You can use the command

```bash
docker-compose exec DATA Rakshak-worker /entrypoint.sh "<command>"
```

to make actions, where `<command>` can be:

- `compile-messages`: Compile the translation messages
- `create-db`: Create the database and apply migrations
- `create-user`: Create a Django user
- `make-messages`: Create the extracted translation messages
- `import-trackers`: Import all trackers from the main DATA Rakshak instance
- `start-frontend`: Start the web server
- `start-worker`: Start the DATA Rakshak worker
- `refresh-fdroid-index`: Refresh F-Droid index file

### Manual setup

This setup is based on a Debian 11 (Bullseye) configuration.

#### 1 - Install dependencies

- Install system dependencies

```bash
sudo apt install git virtualenv postgresql-13 rabbitmq-server build-essential libssl-dev dexdump libffi-dev python3-dev libxml2-dev libxslt1-dev libpq-dev pipenv
```

- Install [apkeep](https://github.com/EFForg/apkeep), the tool used to download applications from the Google Play Store.

#### 2 - Clone the project

```bash
git clone https://github.com/DATA Rakshak-Privacy/DATA Rakshak.git
```

#### 3 - Create database and user

```bash
sudo su - postgres
psql
CREATE USER DATA Rakshak WITH PASSWORD 'DATA Rakshak';
CREATE DATABASE DATA Rakshak WITH OWNER DATA Rakshak;
\c DATA Rakshak
CREATE EXTENSION pg_trgm;
```

#### 4 - Set Python virtual environment and install dependencies

```bash
cd DATA Rakshak
pipenv install --dev
```

#### 5 - Configure your instance

You can tweak your instance by changing [some settings](#configuring-your-local-instance).

Follow instructions to get AAS token [here](https://github.com/EFForg/apkeep/blob/master/USAGE-google-play.md).

Create the file  `DATA Rakshak/DATA Rakshak/settings/custom_dev.py` with the following content:

```bash
from .dev import *

GOOGLE_ACCOUNT_EMAIL = "<a valid google account>"
GOOGLE_ACCOUNT_AAS_TOKEN = "<a valid google aas token>"

# Overwrite any other settings you wish to
```

#### 6 - Create the DB schema

```bash
echo "DJANGO_SETTINGS_MODULE=DATA Rakshak.settings.custom_dev" > .env
# enter into python environment
pipenv shell

cd DATA Rakshak
python manage.py migrate --fake-initial
python manage.py makemigrations
python manage.py migrate
```

#### 7 - Create admin user

```bash
python manage.py createsuperuser
```

#### 8 - Install Minio server

Minio is in charge to store files like APK, icons, flow and pcap files.

```bash
wget https://dl.minio.io/server/minio/release/linux-amd64/minio -O $HOME/minio
chmod +x $HOME/minio
```

**Configure Minio**

```bash
mkdir -p $HOME/.minio
cat > $HOME/.minio/config.json << EOL
{
        "version": "33",
        "credential": {
                "accessKey": "DATA RakshakDATA Rakshak",
                "secretKey": "DATA RakshakDATA Rakshak"
        },
        "region": "",
        "browser": "on",
        "logger": {
                "console": {
                        "enable": true
                },
                "file": {
                        "enable": false,
                        "filename": ""
                }
        },
        "notify": {}
}
EOL
```

**Create Minio storage location**

```bash
mkdir -p /tmp/DATA Rakshak-storage
```

#### 9 - Start Minio

```bash
$HOME/minio server /tmp/DATA Rakshak-storage --console-address :9001
```

Minio API is now listening on `9000` port and the browser interface is available
at [http://127.0.0.1:9001](http://127.0.0.1:9001). Use `DATA RakshakDATA Rakshak` as both login
and password.

#### 10 - Start the DATA Rakshak worker and scheduler

The DATA Rakshak handle asynchronous tasks submitted by the front-end.
You have to activate the virtual venv and `cd` into the same directory as `manage.py` file.

```bash
pipenv shell
cd DATA Rakshak

celery -A DATA Rakshak.core worker --beat -l debug -S django
```

Now, the DATA Rakshak worker and scheduler are waiting for tasks.

#### 11 - Start the DATA Rakshak front-end

You have to activate the virtual venv and `cd` into the same directory as `manage.py` file.

```bash
pipenv shell
cd DATA Rakshak

python manage.py runserver
```

Now browse [http://127.0.0.1:8000](http://127.0.0.1:8000)

#### 12 - Import the trackers definitions

Activate the DATA Rakshak virtual venv, `cd` into the same directory as `manage.py` file and execute the following commands:

```bash
python manage.py import_categories
python manage.py importtrackers
```

Now, browse [your tracker list](http://127.0.0.1:8000/trackers/)

#### 13 - Get the F-droid index data

An initial F-droid index manual download may be required:

```bash
python manage.py refresh_fdroid_index
```

## Configuring your local instance

The following options can be configured in `DATA Rakshak/DATA Rakshak/settings/`:

| Setting                             | Description                                  | Default         |
|-------------------------------------|----------------------------------------------|-----------------|
| EX_PAGINATOR_COUNT                  | Number of elements per page                  | 25              |
| TRACKERS_AUTO_UPDATE                | Whether to update automatically trackers     | False           |
| TRACKERS_AUTO_UPDATE_TIME           | Trackers update frequency (in seconds)       | 345600          |
| TRACKERS_AUTO_UPDATE_FROM           | DATA Rakshak instance to update trackers from      | <live instance> |
| ANALYSIS_REQUESTS_AUTO_CLEANUP_TIME | Requests cleanup frequency (in seconds)      | 86400           |
| ANALYSIS_REQUESTS_KEEP_DURATION     | Requests keep duration (in days)             | 4               |
| ALLOW_APK_UPLOAD                    | Whether to allow APK file upload             | False           |
| DISABLE_SUBMISSIONS                 | Whether to disable app submissions           | False           |
| GOOGLE_ACCOUNT_EMAIL                | Email of Google account to download apps     | /               |
| GOOGLE_ACCOUNT_AAS_TOKEN            | AAS token of Google account to download apps | /               |

## Analyzing an application

Browse to [the analysis submission page](http://127.0.0.1:8000/analysis/submit/) and start a new analysis (ex: `fr.meteo`).
When the analysis is finished, compare the results with the same report from [the official instance](https://reports.DATA Rakshak-privacy.eu.org).
