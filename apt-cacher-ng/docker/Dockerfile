# syntax=docker/dockerfile:1
FROM ubuntu

VOLUME ["/var/cache/apt-cacher-ng"]

RUN sed -i.bak -s 's%http://archive.ubuntu.com/ubuntu/%http://ftp.jaist.ac.jp/pub/Linux/ubuntu/%g' /etc/apt/sources.list \
 && echo 'APT::Install-Recommends 0;' >> /etc/apt/apt.conf.d/01norecommends \
 && echo 'APT::Install-Suggests 0;' >> /etc/apt/apt.conf.d/01norecommends \
 && apt-get update \
 && apt-get install -y apt-cacher-ng \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

EXPOSE 3142

CMD chmod 777 /var/cache/apt-cacher-ng \
 && /etc/init.d/apt-cacher-ng start \
 && tail -f /var/log/apt-cacher-ng/*
