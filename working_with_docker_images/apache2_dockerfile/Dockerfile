FROM alpine:3.6
LABEL maintainer="Himanshu Patel <py.himanshu.patel@gmail.com>"
RUN apk add --no-cache apache2 \
	&& mkdir -p /run/apache2 && \
	echo "<html><h1>Hello Docker</h1></html>" > /var/www/localhost/htdocs/index.html
EXPOSE 80
ENTRYPOINT [ "/usr/bin/httpd", "-D", "FOREGROUND" ]
