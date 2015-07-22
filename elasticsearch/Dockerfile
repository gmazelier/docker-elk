FROM java:8-jre

RUN apt-get -qq update
RUN DEBIAN_FRONTEND=noninteractive apt-get -qq install supervisor vim wget

RUN wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | apt-key add -
RUN echo "deb http://packages.elastic.co/elasticsearch/1.7/debian stable main" | tee -a /etc/apt/sources.list.d/elasticsearch-1.7.list

RUN apt-get -qq update
RUN DEBIAN_FRONTEND=noninteractive apt-get -qq install elasticsearch

RUN rm -rf /var/lib/apt/lists/*

COPY config /usr/share/elasticsearch/config
COPY supervisor /etc/supervisor

ENV ES_HOME /usr/share/elasticsearch

# FIXME Volumes
#VOLUME /var/data/elasticsearch
#VOLUME /var/log/elasticsearch
#VOLUME /tmp/elasticsearch

EXPOSE 9200
EXPOSE 9300
CMD ["supervisord", "-c", "/etc/supervisor/supervisord.conf"]
