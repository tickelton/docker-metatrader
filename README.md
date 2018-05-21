docker-metatrader
=================

Since MetaTrader does not work equally well with all versions of wine and 
installing several versions and switching between them can be a
rather unpleasant experience, running MetaTrader with wine in a Docker
container is a very convenient solution.

Requirements
------------

The image built from this Dockerfile only provides a wine
installation compatible with MetaTrader 4 and MetaTrader 5 and the entry point to run MetaTrader automatically.

MetaTrader itself is not included and has to be supplied
separately.

The MetaTrader directory needed by the image can be acquired either by installing MetaTrader on a Windows machine and copying the installation directory (usually C:\Program\ Files\MetaTrader[45]) or by running the installer in a container and retrieving it from there (see **MetaTrader installation using a container**).

Usage
-----

**Note 1: It is assumed that there is a MetaTrader installation in the host directory $HOME/MetaTrader/.**

**Note 2: The container runs as an unprivileged user with UID/GID 1000/1000. This UID needs to have write access to the host directory containing your MetaTrader installation. If this is not the case, you have to change the IDs in the Dockerfile before building the image.**


Building the image with:
```shell
# docker build .
```
will result in an image containing a suitable installation of Wine but missing the gecko and mono packages.

Therefore if this image is used as is, it will work just fine but install those packages on every start of a container.

This issue can be solved as follow:

* start a container:
```shell
# docker build .
```
* let wine automatically install the missing packages.
* stop the container without any further actions (by closing MetaTrader).
* commit the changes to a new image, e.g.:
```# shell docker commit cc349c658225 mt```

The resulting image **"mt"** will now start MetaTrader immediately without any further prompts.

EXAMPLES
--------

## Building the image:

```shell
# git clone https://github.com/tickelton/docker-metatrader.git
# cd docker-metatrader
# docker build .
```

## Starting a container
```shell
# docker run \
        --net host \
        -v /tmp/.X11-unix:/tmp/.X11-unix \
        -e DISPLAY \
        -v $METATRADER_HOST_PATH:/MetaTrader \
        --name mt \
        tickelton/mt
```

MetaTrader installation using a container
-----------------------------------------

If you do not have a working MetaTrader installation that you can make available in the container, it is possible to install MetaTrader using this very image and retrieving the installation directory from there.

The process for this is as follows:

* build the image as described in **Building the image**.
* create a temporary directory on your host, that  MetaTrader is going to be installed in. 
```shell
# mkdir /tmp/mt
```
* download the metatrader installer (mt4setup.exe or mt5setup.exe) from [MetaQuotes](https://www.metaquotes.net/) and place it in /tmp/mt.
* start the container with the install directory mounted to /home/wine and bash as entry point:
```shell
# docker run \
	-ti \
        --net host \
        -v /tmp/.X11-unix:/tmp/.X11-unix \
        -e DISPLAY \
        -v /tmp/mt:/home/wine \
        --entrypoint=/bin/sh \
        tickelton/mt
```
* install MetaTrader:
```shell
# wine /home/wine/mt5setup.exe
```
* MetaTrader is now installed into /tmp/mt/.wine/drive_c/Program Files/MetaTrader 5/
* you can now save this directory and run your dockerized MetaTrader from there:
```shell
# cp -r /tmp/mt/.wine/drive_c/Program Files/MetaTrader 5/ $HOME/MetaTrader
...
# docker run \
        --net host \
        -v /tmp/.X11-unix:/tmp/.X11-unix \
        -e DISPLAY \
        -v $HOME/MetaTrader:/MetaTrader \
        --name mt \
        tickelton/mt
```

