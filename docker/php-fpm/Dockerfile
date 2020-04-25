FROM alpine:3.11
RUN apk add --update \
--repository https://sjc.edge.kernel.org/alpine/v3.8/main \
--repository https://sjc.edge.kernel.org/alpine/v3.8/community \
php5-fpm && rm -rf /var/cache/apk/*
RUN ln -s /dev/stdout /var/log/php-fpm.log
EXPOSE 9000
CMD [ "/usr/bin/php-fpm5", "--nodaemonize"]
