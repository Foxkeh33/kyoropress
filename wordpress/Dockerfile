FROM wordpress:latest

# WP CLI INSTALL
RUN curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar \
    && chmod +x wp-cli.phar \
    && mv wp-cli.phar /usr/local/bin/wp \
    && wp --info

# PHP INSTALL
RUN apt-get update \
    && apt-get install -y \
    vim \
    git \
    wget \
    unzip

ADD ssl.conf /etc/apache2/sites-available/ssl.conf
RUN a2enmod ssl
RUN a2ensite ssl