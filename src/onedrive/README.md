# Run the OneDrive Client for Linux under Docker

## Run or update with one script

If you are experienced with docker and onedrive, you can use the following script:

```bash
# Update ONEDRIVE_DATA_DIR with correct OneDrive directory path
export ONEDRIVE_UID=`id -u`
export ONEDRIVE_GID=`id -g`
export ONEDRIVE_DATA_DIR="${HOME}/workspace/onedrive"
export ONEDRIVE_CONF_DIR="${HOME}/.config/onedrive"
# Create directory if non-existant
mkdir -p ${ONEDRIVE_DATA_DIR}
mkdir -p ${ONEDRIVE_CONF_DIR}

dkcr onedrive onedrive
```

**Important:** The 'target' folder of `ONEDRIVE_DATA_DIR` must exist before running the Docker container, otherwise, Docker will create the target folder, and the folder will be given 'root' permissions, which then causes the Docker container to fail upon startup with the following error message:

```bash
ROOT level privileges prohibited!
```

**NOTE:** It is also highly advisable for you to replace `${ONEDRIVE_UID}` and `${ONEDRIVE_GID}` with your actual UID and GID as specified by your `id` command output to avoid any any potential user or group conflicts.

When the Docker container successfully starts:

- You will be asked to open a specific link using your web browser
- Login to your Microsoft Account and give the application the permission
- After giving the permission, you will be redirected to a blank page
- Copy the URI of the blank page into the application prompt to authorise the application

Once the 'onedrive' application is authorised, the client will automatically start monitoring your `ONEDRIVE_DATA_DIR` for data changes to be uploaded to OneDrive. Files stored on OneDrive will be downloaded to this location.

If the client is working as expected, you can detach from the container with Ctrl+p, Ctrl+q.

## Docker Container Status, stop, and restart

Check if the monitor service is running

```bash
docker ps -f name=onedrive
```

Show monitor run logs

```bash
docker logs onedrive
```

Stop running monitor

```bash
docker stop onedrive
```

Resume monitor

```bash
docker start onedrive
```

Remove onedrive Docker container

```bash
docker rm -f onedrive
```

## Advanced Setup

### 5. Docker-compose

Also supports docker-compose schemas > 3.
In the following example it is assumed you have a `ONEDRIVE_DATA_DIR` environment variable and a `onedrive_conf` volume.
However, you can also use bind mounts for the configuration folder, e.g. `export ONEDRIVE_CONF="${HOME}/OneDriveConfig"`.

```
version: "3"
services:
    onedrive:
        image: driveone/onedrive:latest
        restart: unless-stopped
        environment:
            - ONEDRIVE_UID=${PUID}
            - ONEDRIVE_GID=${PGID}
        volumes:
            - onedrive_conf:/onedrive/conf
            - ${ONEDRIVE_DATA_DIR}:/onedrive/data
```

Note that you still have to perform step 3: First Run.

## Edit the config

The 'onedrive' client should run in default configuration, however you can change this default configuration by placing a custom config file in the `onedrive_conf` docker volume. First download the default config from [here](https://raw.githubusercontent.com/abraunegg/onedrive/master/config)  
Then put it into your onedrive_conf volume path, which can be found with:

```bash
docker volume inspect onedrive_conf
```

Or you can map your own config folder to the config volume. Make sure to copy all files from the docker volume into your mapped folder first.

The detailed document for the config can be found here: [Configuration](https://github.com/abraunegg/onedrive/blob/master/docs/USAGE.md#configuration)

## Sync multiple accounts

There are many ways to do this, the easiest is probably to

1. Create a second docker config volume (replace `Work` with your desired name): `docker volume create onedrive_conf_Work`
2. And start a second docker monitor container (again replace `Work` with your desired name):

```
export ONEDRIVE_DATA_DIR_WORK="/home/abraunegg/OneDriveWork"
mkdir -p ${ONEDRIVE_DATA_DIR_WORK}
docker run -it --restart unless-stopped --name onedrive_Work -v onedrive_conf_Work:/onedrive/conf -v "${ONEDRIVE_DATA_DIR_WORK}:/onedrive/data" driveone/onedrive:latest
```

## Environment Variables

| Variable                         | Purpose                                                                   |                                                           Sample Value                                                           |
| -------------------------------- | ------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------: |
| <B>ONEDRIVE_UID</B>              | UserID (UID) to run as                                                    |                                                               1000                                                               |
| <B>ONEDRIVE_GID</B>              | GroupID (GID) to run as                                                   |                                                               1000                                                               |
| <B>ONEDRIVE_VERBOSE</B>          | Controls "--verbose" switch on onedrive sync. Default is 0                |                                                                1                                                                 |
| <B>ONEDRIVE_DEBUG</B>            | Controls "--verbose --verbose" switch on onedrive sync. Default is 0      |                                                                1                                                                 |
| <B>ONEDRIVE_DEBUG_HTTPS</B>      | Controls "--debug-https" switch on onedrive sync. Default is 0            |                                                                1                                                                 |
| <B>ONEDRIVE_RESYNC</B>           | Controls "--resync" switch on onedrive sync. Default is 0                 |                                                                1                                                                 |
| <B>ONEDRIVE_DOWNLOADONLY</B>     | Controls "--download-only" switch on onedrive sync. Default is 0          |                                                                1                                                                 |
| <B>ONEDRIVE_UPLOADONLY</B>       | Controls "--upload-only" switch on onedrive sync. Default is 0            |                                                                1                                                                 |
| <B>ONEDRIVE_LOGOUT</B>           | Controls "--logout" switch. Default is 0                                  |                                                                1                                                                 |
| <B>ONEDRIVE_REAUTH</B>           | Controls "--reauth" switch. Default is 0                                  |                                                                1                                                                 |
| <B>ONEDRIVE_AUTHFILES</B>        | Controls "--auth-files" option. Default is ""                             |                                                      "authUrl:responseUrl"                                                       |
| <B>ONEDRIVE_AUTHRESPONSE</B>     | Controls "--auth-response" option. Default is ""                          | See [here](https://github.com/abraunegg/onedrive/blob/master/docs/USAGE.md#authorize-the-application-with-your-onedrive-account) |
| <B>ONEDRIVE_DISPLAY_CONFIG</B>   | Controls "--display-running-config" switch on onedrive sync. Default is 0 |                                                                1                                                                 |
| <B>ONEDRIVE_SINGLE_DIRECTORY</B> | Controls "--single-directory" option. Default = ""                        |                                                             "mydir"                                                              |

### Usage Examples

**Verbose Output:**

```bash
docker container run -e ONEDRIVE_VERBOSE=1 -v onedrive_conf:/onedrive/conf -v "${ONEDRIVE_DATA_DIR}:/onedrive/data" driveone/onedrive:latest
```

**Debug Output:**

```bash
docker container run -e ONEDRIVE_DEBUG=1 -v onedrive_conf:/onedrive/conf -v "${ONEDRIVE_DATA_DIR}:/onedrive/data" driveone/onedrive:latest
```

**Perform a --resync:**

```bash
docker container run -e ONEDRIVE_RESYNC=1 -v onedrive_conf:/onedrive/conf -v "${ONEDRIVE_DATA_DIR}:/onedrive/data" driveone/onedrive:latest
```

**Perform a --resync and --verbose:**

```bash
docker container run -e ONEDRIVE_RESYNC=1 -e ONEDRIVE_VERBOSE=1 -v onedrive_conf:/onedrive/conf -v "${ONEDRIVE_DATA_DIR}:/onedrive/data" driveone/onedrive:latest
```

**Perform a --logout and re-authenticate:**

```bash
docker container run -it -e ONEDRIVE_LOGOUT=1 -v onedrive_conf:/onedrive/conf -v "${ONEDRIVE_DATA_DIR}:/onedrive/data" driveone/onedrive:latest
```

## Build instructions

#### How to support double-byte languages

In some geographic regions, you may need to change and/or update the locale specification of the Docker container to better support the local language used for your local filesystem. To do this, follow the example below:

```
FROM driveone/onedrive

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update
RUN apt-get install -y locales

RUN echo "ko_KR.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen ko_KR.UTF-8 && \
    dpkg-reconfigure locales && \
    /usr/sbin/update-locale LANG=ko_KR.UTF-8

ENV LC_ALL ko_KR.UTF-8
```

The above example changes the Docker container to support Korean. To support your local language, change `ko_KR.UTF-8` to the required entry.
