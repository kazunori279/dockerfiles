# Dockerfile for fluentd + nginx
#
# Usage:
# sudo docker run -e NORIKRA_IP=<<YOUR NORIKRA SERVER IP>> -p 80:80 -t -i -d kazunori279/fluentd-nginx-bq 

FROM kazunori279/fluentd-norikra
MAINTAINER kazunori279-at-gmail.com

# nginx
RUN apt-get install -y nginx
ADD nginx.conf /etc/nginx/nginx.conf

# fluent-plugin-dstat 
RUN apt-get install dstat
RUN /usr/lib/fluent/ruby/bin/fluent-gem install fluent-plugin-dstat --no-ri --no-rdoc -V

# fluent-plugin-bigquery
RUN /usr/lib/fluent/ruby/bin/fluent-gem install fluent-plugin-bigquery --no-ri --no-rdoc -V

# td-agent.conf
ADD td-agent.conf /etc/td-agent/td-agent.conf.tmp

# start and open services
EXPOSE 80
ENTRYPOINT sed -e "s/%%NORIKRA_IP%%/$NORIKRA_IP/" /etc/td-agent/td-agent.conf.tmp > /etc/td-agent/td-agent.conf && norikra start --more-verbose -Xmx2g --pidfile /var/run/norikra.pid --logdir=/var/log/norikra --daemonize && /etc/init.d/td-agent restart && /etc/init.d/nginx start && /bin/bash 

