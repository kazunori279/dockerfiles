FROM ubuntu:12.04
MAINTAINER kazunori279@gmail.com

# environment
ENV DEBIAN_FRONTEND noninteractive
RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list

# OpenJDK, curl, gcc
RUN apt-get update && apt-get -y upgrade
RUN apt-get -y install openjdk-7-jre-headless curl build-essential
ENV JAVA_HOME /usr/lib/jvm/java-7-openjdk-amd64

# jruby
RUN curl -O http://jruby.org.s3.amazonaws.com/downloads/1.7.10/jruby-bin-1.7.10.tar.gz
RUN tar xzf jruby-bin-1.7.10.tar.gz && mkdir -p /usr/lib/jruby && mv jruby-1.7.10 /usr/lib/jruby
ENV PATH /usr/lib/jruby/jruby-1.7.10/bin:$PATH

# norikra
RUN gem install norikra --no-ri --no-rdoc -V
RUN mkdir -p /var/log/norikra && norikra start -Xmx2g --daemonize --pidfile /var/run/norikra.pid --logdir=/var/log/norikra

# fluentd
RUN curl -O http://packages.treasure-data.com/debian/RPM-GPG-KEY-td-agent && apt-key add RPM-GPG-KEY-td-agent && rm RPM-GPG-KEY-td-agent
RUN curl -L http://toolbelt.treasuredata.com/sh/install-ubuntu-precise.sh | sh

# fluent-plugin-norikra
RUN /usr/lib/fluent/ruby/bin/fluent-gem install fluent-plugin-norikra --no-ri --no-rdoc -V

# start and open services
EXPOSE 26578
EXPOSE 26571
ENTRYPOINT /etc/init.d/td-agent restart && norikra start -Xmx2g --pidfile /var/run/norikra.pid --logdir=/var/log/norikra

