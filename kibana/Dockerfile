FROM nginx

RUN apt-get -qq update
RUN DEBIAN_FRONTEND=noninteractive apt-get -qq install supervisor vim wget

RUN rm -rf /var/lib/apt/lists/*

RUN mkdir -p /usr/share/kibana
RUN wget --no-check-certificate -q -O- https://download.elastic.co/kibana/kibana/kibana-4.1.1-linux-x64.tar.gz | tar xfz - -C /usr/share/kibana --strip-components 1

COPY nginx /etc/nginx
COPY supervisor /etc/supervisor
COPY kibana /usr/share/kibana

# ENV KIBANA_SECURE true
# ENV KIBANA_USER kibana
# ENV KIBANA_PASSWORD kibana

EXPOSE 80
CMD ["supervisord", "-c", "/etc/supervisor/supervisord.conf"]
