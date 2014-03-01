# Dockerfile for fluentd + norikra server
#
# Usage:
# $ sudo docker run -p 26578:26578 -p 26571:26571 -p 24224:24224 -p 24224:24224/udp -d kazunori279/fluentd-norikra-server
#

FROM kazunori279/fluentd-norikra 
MAINTAINER kazunori279-at-gmail.com

# start and open services
EXPOSE 26578
EXPOSE 26571
EXPOSE 24224
ENTRYPOINT /etc/init.d/td-agent restart && norikra start -Xmx2g --pidfile /var/run/norikra.pid --logdir=/var/log/norikra

