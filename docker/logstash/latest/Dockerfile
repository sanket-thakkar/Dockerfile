#+++++++++++++++++++++++++++++++++++++++
# Dockerfile for webdevops/logstash:latest
#    -- automatically generated  --
#+++++++++++++++++++++++++++++++++++++++

FROM webdevops/base-app:alpine-3

MAINTAINER info@webdevops.io
LABEL vendor=WebDevOps.io
LABEL io.webdevops.layout=8
LABEL io.webdevops.version=0.56.0

ENV LOGSTASH_VERSION=2.4.0

RUN addgroup logstash \
    && mkdir -p /var/lib/logstash \
    && adduser -S -D -g "" -G logstash -s /sbin/nologin -h /var/lib/logstash logstash \
    && chown -R logstash:logstash /var/lib/logstash \
    && /usr/local/bin/apk-install \
        openjdk8-jre \
    && curl -sSL https://download.elastic.co/logstash/logstash/logstash-${LOGSTASH_VERSION}.tar.gz | tar -xz -C /usr/local \
    && ln -s /usr/local/logstash-${LOGSTASH_VERSION} /usr/local/logstash \
    && rm -fr /usr/local/logstash/bin/*.bat

RUN /usr/local/logstash/bin/logstash-plugin install \
    logstash-output-redmine \
    logstash-output-influxdb \
    logstash-output-mongodb \
    logstash-filter-aggregate \
    logstash-filter-alter \
    logstash-filter-prune
