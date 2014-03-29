# Dockerfile for fluentd + nginx
#
# Usage:
# sudo docker run -e NORIKRA_IP=<<YOUR NORIKRA SERVER IP>> -p 80:80 -t -i -d kazunori279/fluentd-nginx 

FROM kazunori279/fluentd 
MAINTAINER kazunori279-at-gmail.com

# nginx
RUN apt-get install -y nginx
ADD nginx.conf /etc/nginx/nginx.conf

# fluent-plugin-dstat 
RUN apt-get install dstat
RUN /usr/lib/fluent/ruby/bin/fluent-gem install fluent-plugin-dstat --no-ri --no-rdoc -V

# td-agent.conf
ADD td-agent.conf /etc/td-agent/td-agent.conf.tmp

# start and open services
EXPOSE 80
ENTRYPOINT sed -e "s/%%NORIKRA_IP%%/$NORIKRA_IP/" /etc/td-agent/td-agent.conf.tmp > /etc/td-agent/td-agent.conf && /etc/init.d/td-agent restart && /etc/init.d/nginx start && /bin/bash 

