# Vackup2: Backup and Restore Docker Volumes

Continuation of [Bret Fisher's docker-vackup](https://github.com/BretFisher/docker-vackup) which he discontinued as of 2022 Sep 2.
Renamed the repo as vackup2 with "2" as in 2nd version, after the main tool that does the actual backup.

Yes, this is a fork of mentioned tool but decided not to use github's fork option as I don't want this repo to get deleted if in
the future, Bret decides to delete the old repo for some reason.

Vackup2: (contraction of "volume backup")

Easily backup and restore Docker volumes using either tarballs or container images.
It's designed for running from any host/container where you have the docker CLI.

Note that for open files like databases,
it's usually better to use their preferred backup tool to create a backup file,
but if you stored that file on a Docker volume,
this could still be a way you get the Docker volume into a image or tarball
for moving to remote storage for safe keeping.

`export`/`import` commands copy files between a local tarball and a volume.
For making volume backups and restores.

`save`/`load` commands copy files between an image and a volume.
For when you want to use image registries as a way to push/pull volume data.

Usage:

`vackup2 export VOLUME FILE`
  Creates a gzip'ed tarball in current directory from a volume

`vackup2 import FILE VOLUME`
  Extracts a gzip'ed tarball into a volume

`vackup2 save VOLUME IMAGE`
  Copies the volume contents to a busybox image in the /volume-data directory

`vackup2 load IMAGE VOLUME`
  Copies /volume-data contents from an image to a volume

## Backup and restore all volumes

Backup

```
for vl in $(docker volume ls | awk '{print $2 }'); do vackup2 export $vl ${vl}.tar.gz ; done
```

Restore

```
for fl in $(ls *.tar.gz); do vackup2 import $fl ${fl%%.*} ; done
```

## Install

Download the `vackup2` file in this repository to your local machine in your shell path and make it executable.

```shell
curl -o /usr/local/bin/vackup2 -sSL https://raw.githubusercontent.com/icasimpan/vackup2/main/vackup2
chmod +x /usr/local/bin/vackup2
```

## Error conditions

If any of the commands fail, the script will check to see if a `VACKUP2_FAILURE_SCRIPT`
environment variable is set.  If so it will run it and pass the line number the error
happened on and the exit code from the failed command.  Eg,

```shell
# /opt/bin/vackup2-failed.sh
LINE_NUMBER=$1
EXIT_CODE=$2
send_slack_webhook "Vackup2 failed on line number ${LINE_NUMBER} with exit code ${EXIT_CODE}!"
```

```shell
export VACKUP2_FAILURE_SCRIPT=/opt/bin/vackup2-failed.sh
./vackup2 export ......
```
