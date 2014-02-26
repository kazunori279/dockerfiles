FROM kazunori279/fluentd-norikra 
MAINTAINER kazunori279@gmail.com

# start and open services
EXPOSE 26578
EXPOSE 26571
ENTRYPOINT /etc/init.d/td-agent restart && norikra start -Xmx2g --pidfile /var/run/norikra.pid --logdir=/var/log/norikra

