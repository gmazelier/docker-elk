#ELK#

This project defines and runs a complete ELK environment as a multi-container application.
Usage describes a Maven based Java application configuration but other JVM based languages such as Groovy, Clojure or Scala support should be straightforward.

##Context##

|Application|Base container|Version|Exposed ports|
|-----------|--------------|------:|------------:|
|Elasticsearch|`java:8-jre`|1.7|9200,9300|
|Logstash|`java:8-jre`|1.5|55444 (TCP/UDP)|
|Kibana|`nginx`|4.1.1|80|

##Usage##

The `docker-compose` following configurations files are used:

* `docker-compose-base.yml` defines the different containers and their properties, e.g. the exposed ports forwarding or the mounted volumes,
* `docker-compose-build.yml` provides a way to build the required images,
* the default file `docker-compose.yml` is used to run the application.

###Building####

To build the required images, use the following command:

    docker-compose -f docker-compose-build.yml build

###Running####

With the images available, use the standard way to launch the containers, e.g.:

    docker-compose up

###Feeding Logstash###

Now ELK is running, you need to feed Logstash before using Kibana to explore and visualize your data.
The Logstash container is configured to accept [logstash-forwarder](https://github.com/elastic/logstash-forwarder) inputs.

The following example uses [logstash-logback-encoder](https://github.com/logstash/logstash-logback-encoder) to send data to Logstash from a Maven based Java application with [Logback](http://logback.qos.ch) as logging library.

1. Check your SL4J, Logback, Jackson Maven dependencies. See [logstash-forwarder](https://github.com/elastic/logstash-forwarder) for more information about the requirements.

1. Add `logstash-forwarder` dependency to your Maven POM:

        <dependency>
            <groupId>net.logstash.logback</groupId>
            <artifactId>logstash-logback-encoder</artifactId>
            <version>4.4</version>
        </dependency>

1. Configure a `RollingFileAppender` appender using a `LogstashEncoder` encoder in you `logback.xml` configuration file:

        <appender name="JSON" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>${json.log.path}.log</file>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>${json.log.path}.%d{yyyy-MM-dd}.log</fileNamePattern>
                <maxHistory>${json.max.history}</maxHistory>
            </rollingPolicy>
            <encoder class="net.logstash.logback.encoder.LogstashEncoder" />
            <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
                <level>${json.appender.level}</level>
            </filter>
        </appender>

1. Configure `logstash-forwarder`. The SSL key and CA are those available in the Logstash container [directory](logstash/).
Check the *Misc* section below for more information about the certificate generation.
The name `logstash.example.com` should be an alias to your Docker host IP.
Your `logstash-forwarder.conf` file should look like this:

        {
            "network": {
                "servers": [ "logstash.example.com:55444" ],
                "ssl key": "./lumberjack.key",
                "ssl ca": "./lumberjack.crt",
                "timeout": 15
            },
            "files": [
                {
                    "paths": [
                        "/some/path/myapp_*.log"
                    ],
                    "fields": { "type": "myapp" }
                }
            ]
        }

1. Start `logstash-forwarder`:

        ./logstash-forwarder -config=logstash-forwarder.conf

1. Start your Java application. Produced logs should by emitted by `logstash-forwarder` to Logstash.

###Accessing Kibana###

Update your `/etc/hosts` file to create an alias `kibana` to your Docker host IP.
This value depends on the way your run Docker : `boot2docker`, `docker-machine`, Linux daemon, etc.
The following command prints the IP address of your `MACHINE`, managed by `docker-machine`:

    docker-machine ip MACHINE

Browse now to [Kibana](http://kibana:80).

##Misc##

###Logstash certificate###

This command is used to generate your own certificate with a wildcard Common Name `*.example.com`:

    openssl req -x509 -nodes -newkey rsa:2048 \
      -keyout lumberjack.key -out lumberjack.crt \
      -subj /CN=*.example.com
