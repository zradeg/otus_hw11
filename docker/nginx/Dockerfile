FROM alpine:3.11
RUN apk update && apk add nginx && \
	mkdir -p /usr/share/nginx/html && \
	ln -s /dev/stdout /var/log/nginx/error.log & \
	ln -s /dev/stdout /var/log/nginx/access.log
EXPOSE 80
CMD [ "nginx", "-g", "daemon off;" ]
