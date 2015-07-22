FROM java:8-jre

RUN apt-get -qq update
RUN DEBIAN_FRONTEND=noninteractive apt-get -qq install vim wget

RUN wget -qO - https://packages.elasticsearch.org/GPG-KEY-elasticsearch | apt-key add -
RUN echo "deb http://packages.elasticsearch.org/logstash/1.5/debian stable main" | tee -a /etc/apt/sources.list

RUN apt-get -qq update
RUN DEBIAN_FRONTEND=noninteractive apt-get -qq install logstash

RUN rm -rf /var/lib/apt/lists/*

COPY logstash.conf /etc/logstash.conf
COPY lumberjack.crt /etc/lumberjack.crt
COPY lumberjack.key /etc/lumberjack.key

EXPOSE 55444
CMD ["/opt/logstash/bin/logstash", "-f", "/etc/logstash.conf"]
