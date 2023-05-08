# vim: set tabstop=8 noexpandtab softtabstop=0 shiftwidth=0 :

FROM fluentd:v1.16.0-1.0

ARG FLUENTD_PLUGINS
ARG FLUENTD_UID=1000
ARG FLUENTD_DIR=/home/fluser

USER root

RUN apk add --no-cache --update --virtual .build-deps sudo build-base ruby-dev \
 && for n in ${FLUENTD_PLUGINS}; do \
      gem install $n; \
    done \
 && gem sources --clear-all \
 && apk del .build-deps \
 && rm -rf /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem \
 && adduser -u ${FLUENTD_UID} -h ${FLUENTD_DIR} -G fluent -D fluser

USER fluser
