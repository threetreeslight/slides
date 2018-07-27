FROM debian:latest

RUN apt-get update && apt-get install -y binutils debootstrap
RUN mkdir -p /srv/chroot/wheezy
RUN debootstrap --arch i386 wheezy /srv/chroot/wheezy http://httpredir.debian.org/debian

RUN apt-get install -y systemd-container
CMD sh
