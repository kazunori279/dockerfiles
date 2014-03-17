# Dockerfile for fluentd + norikra server + Google Spreadsheet
#
# Usage:
# $ sudo docker run -p 26578:26578 -p 26571:26571 -p 24224:24224 -p 24224:24224/udp -e GAS_URL=<<YOUR GAS URL>> -t -i -d kazunori279/fluentd-norikra-gas

FROM kazunori279/fluentd-norikra 
MAINTAINER kazunori279-at-gmail.com

# td-agent settings
ADD td-agent.conf /etc/td-agent/td-agent.conf.tmp

# https plugin
RUN curl https://raw.github.com/kazunori279/fluent-plugin-out-https/v.0.1.1/lib/fluent/plugin/out_https.rb -o /etc/td-agent/plugin/out_https.rb

# start and open services
EXPOSE 26578
EXPOSE 26571
EXPOSE 24224
ENTRYPOINT sed -e "s@%%GAS_URL%%@$GAS_URL@" /etc/td-agent/td-agent.conf.tmp > /etc/td-agent/td-agent.conf && norikra start -Xmx2g --pidfile /var/run/norikra.pid --logdir=/var/log/norikra --daemonize && /etc/init.d/td-agent restart && /bin/bash 
