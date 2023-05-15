FROM ubuntu:20.04

LABEL maintainer="Glen Bangkila"

RUN useradd -ms /bin/bash -u 1337 dev
WORKDIR /var/www/html

# Avoid prompts when building
ENV DEBIAN_FRONTEND noninteractive
ENV TZ=UTC
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update \
    && apt-get install -y gnupg gosu curl ca-certificates zip unzip git supervisor sqlite3 \
    && mkdir -p ~/.gnupg \
    && echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf \
    && apt-key adv --homedir ~/.gnupg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys E5267A6C \
    && apt-key adv --homedir ~/.gnupg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C300EE8C \
    && echo "deb http://ppa.launchpad.net/ondrej/php/ubuntu focal main" > /etc/apt/sources.list.d/ppa_ondrej_php.list \
    && apt-get update \
    && apt-get install -y nginx php7.3-fpm php7.3-cli \
       php7.3-pgsql php7.3-sqlite3 php7.3-gd \
       php7.3-curl php7.3-memcached \
       php7.3-imap php7.3-mysql php7.3-mbstring \
       php7.3-xml php7.3-zip php7.3-bcmath php7.3-soap \
       php7.3-intl php7.3-readline php7.3-pcov \
       php7.3-msgpack php7.3-igbinary php7.3-ldap \
       php7.3-redis \
    && php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer --version=1.10.24 \
    && apt-get -y autoremove \
    && sed -i "s/pm\.max_children = .*/pm.max_children = 20/" /etc/php/7.3/fpm/pool.d/www.conf \
    && sed -i "s/pm\.start_servers = .*/pm.start_servers = 10/" /etc/php/7.3/fpm/pool.d/www.conf \
    && sed -i "s/pm\.min_spare_servers = .*/pm.min_spare_servers = 5/" /etc/php/7.3/fpm/pool.d/www.conf \
    && sed -i "s/pm\.max_spare_servers = .*/pm.max_spare_servers = 10/" /etc/php/7.3/fpm/pool.d/www.conf \
    && echo "daemon off;" >> /etc/nginx/nginx.conf \
    && ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log \
    && sed -i 's/^;daemonize.*$/daemonize = no/g' /etc/php/7.3/fpm/php-fpm.conf \
    && sed -i 's@^error_log.*$@error_log = /proc/self/fd/2@g' /etc/php/7.3/fpm/php-fpm.conf \
    && apt-get install -y mysql-client \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY start /usr/local/bin/start
COPY h5bp /etc/nginx/h5bp
COPY nginx /etc/nginx/sites-available/default
COPY ssh /home/eepower/.ssh
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY php.ini /etc/php/7.3/fpm/conf.d/99-eepower.ini
COPY php.ini /etc/php/7.3/cli/conf.d/99-eepower.ini
RUN chmod +x /usr/local/bin/start

EXPOSE 80

ENTRYPOINT ["start"]