# Dockerfile for fluentd + nginx

FROM kazunori279/fluentd 
MAINTAINER kazunori279-at-gmail.com

# settings
ENV NORIKRA_SERVER_HOST docker-test1

# nginx
RUN apt-get install -y nginx
ADD nginx.conf /etc/nginx/nginx.conf

# td-agent.conf
ADD td-agent.conf /etc/td-agent/td-agent.conf

# start and open services
EXPOSE 80
CMD /etc/init.d/td-agent restart && /etc/init.d/nginx start

