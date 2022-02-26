FROM ubuntu:22.04

# Install apache2
RUN set -eux; \
	apt update && apt install -y --no-install-recommends \
		apache2 \
	;

# Redirect log files to stderr/stdout
RUN ln -sf /proc/self/fd/1 /var/log/apache2/access.log && \
    ln -sf /proc/self/fd/1 /var/log/apache2/error.log

# Copy site configurations into container
COPY config/ /etc/apache2/sites-available/

# Copy site configurations into container
COPY html/ /var/www/html/

EXPOSE 80

ENTRYPOINT ["apachectl", "-D", "FOREGROUND"]