FROM php:7.4-fpm
LABEL maintainer="Shahriyar Imanov <shehi@imanov.me>"

WORKDIR /var/www

ENV DEBIAN_FRONTEND noninteractive
ENV TERM xterm

RUN apt-get update -y \
    && apt-get install -y apt-utils \
    && apt-get upgrade -y \
    && mkdir -p /usr/share/man/man1 \
    && apt-get install -y \
        bzip2 \
        cron \
        daemon \
        fontforge \
        git \
        gnupg \
        iproute2 \
        iputils-ping \
        locales \
        locales-all \
        mariadb-client \
        nano \
        openjdk-11-jre \
        procps \
        rsyslog \
        sudo \
        telnet \
        unzip \
        wget

# Locale
RUN locale-gen en_US.UTF-8 && locale-gen de_DE.UTF-8 && dpkg-reconfigure locales
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

# Install PHP extensions
RUN docker-php-ext-install \
        bcmath \
        calendar \
        exif \
        gettext \
        pdo_mysql \
        sockets

RUN apt-get install -y libfreetype6-dev libjpeg-dev libjpeg62-turbo-dev libpng-dev \
    && docker-php-ext-configure gd \
    && docker-php-ext-install gd

RUN apt-get install -y libicu-dev && docker-php-ext-install intl

RUN apt-get install -y libmcrypt-dev libmemcached-dev \
    && pecl install -of mcrypt memcached \
    && docker-php-ext-enable mcrypt memcached

RUN apt-get install -y libpq-dev && docker-php-ext-install pdo_pgsql

RUN apt-get install -y libxml2-dev && docker-php-ext-install soap

RUN apt-get install -y libxslt1-dev && docker-php-ext-install xsl

RUN apt-get install -y libzip-dev && docker-php-ext-install zip

# Install Google's depot-tools
#RUN apt-get install -y python libglib2.0-dev
#RUN git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git /tmp/.depot-tools
#ENV PATH=$PATH:/tmp/.depot-tools

# Fetch and compile v8
#RUN cd /tmp \
#    && fetch v8 \
#    && cd v8 \
#    && git checkout 6.4.345 \
#    && gclient sync \
#    && tools/dev/v8gen.py -vv x64.release -- is_component_build=true \
#    && ninja -C out.gn/x64.release/ \
#    && mkdir -p /opt/v8/lib \
#    && mkdir -p /opt/v8/include \
#    && cp out.gn/x64.release/lib*.so out.gn/x64.release/*_blob.bin out.gn/x64.release/icudtl.dat /opt/v8/lib/ \
#    && cp -R include/* /opt/v8/include/ \
#    && apt-get remove python -y

# Compile php-v8js
#RUN git clone --depth=1 https://github.com/phpv8/v8js.git /tmp/v8js \
#    && cd /tmp/v8js \
#    && phpize \
#    && ./configure --with-v8js=/opt/v8 LDFLAGS="-lstdc++"
#ENV NO_INTERACTION 1
#RUN cd /tmp/v8js \
#    && make \
#    && make test \
#    && make install \
#    && docker-php-ext-enable v8js

# Install PECL php-mongodb
RUN pecl install -of mongodb \
    && rm -rf /tmp/pear \
    && echo "extension=mongodb.so" > /usr/local/etc/php/conf.d/mongo.ini

# Install PECL php-xdebug
RUN pecl install -of xdebug \
    && rm -rf /tmp/pear \
    && echo "zend_extension=xdebug.so" | tee /usr/local/etc/php/conf.d/xdebug.ini

# Port for XDebug
EXPOSE 54321

# Timezone
RUN echo "date.timezone = UTC" | tee /usr/local/etc/php/conf.d/date.ini

# Install php-redis extension through Git
RUN git clone https://github.com/phpredis/phpredis.git /tmp/phpredis \
    && cd /tmp/phpredis \
    && git checkout 5.3.1 \
    && phpize \
    && ./configure \
    && make \
    && make install \
    && echo "extension=redis.so" | tee /usr/local/etc/php/conf.d/redis.ini

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer

# Some checks
RUN service rsyslog start && service cron start && service --status-all
RUN php -m

# Install Node
ENV NPM_CONFIG_LOGLEVEL warn
ENV NODE_VERSION 14.15.5
RUN ARCH= && dpkgArch="$(dpkg --print-architecture)" \
    && case "${dpkgArch##*-}" in \
        amd64) ARCH='x64';; \
        ppc64el) ARCH='ppc64le';; \
        s390x) ARCH='s390x';; \
        arm64) ARCH='arm64';; \
        armhf) ARCH='armv7l';; \
        i386) ARCH='x86';; \
        *) echo "unsupported architecture"; exit 1 ;; \
        esac \
    # gpg keys listed at https://github.com/nodejs/node#release-keys
    && set -ex \
    && for key in \
        4ED778F539E3634C779C87C6D7062848A1AB005C \
        94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
        74F12602B6F1C4E913FAA37AD3A89613643B6201 \
        71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
        8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 \
        C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
        C82FA3AE1CBEDC6BE46B9360C43CEC45C17AB93C \
        DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
        A48C2BEE680E841632CD4E44F07496B3EB3C1762 \
        108F52B48DB57BB0CC439B2997B01419BD92F80A \
        B9E2F5981AA6E0CD28160D9FF13993A75599653C \
    ; do \
        gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || \
        gpg --batch --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
        gpg --batch --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; \
    done \
    && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-$ARCH.tar.xz" \
    && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
    && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
    && grep " node-v$NODE_VERSION-linux-$ARCH.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
    && tar -xJf "node-v$NODE_VERSION-linux-$ARCH.tar.xz" -C /usr/local --strip-components=1 --no-same-owner \
    && rm "node-v$NODE_VERSION-linux-$ARCH.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
    && ln -s /usr/local/bin/node /usr/local/bin/nodejs \
    # smoke tests
    && node --version \
    && npm --version \
    && rm -rf /tmp/npm-*

# Install Node global dependencies
RUN npm install -g ttf2eot jshint webpack webpack-cli mocha

# Port for node --inspect
EXPOSE 9229

# Install fonting tools
RUN apt-get update -y && apt-get install woff-tools \
    && git clone --depth=1 https://github.com/google/woff2.git /opt/woff2 \
    && cd /opt/woff2 \
    && git submodule init \
    && git submodule update \
    && make clean all \
    && chmod +x /opt/woff2/woff2_* \
    && git clone --depth=1 https://github.com/zoltan-dulac/css3FontConverter.git /opt/css3_font_converter \
    && chmod +x /opt/css3_font_converter/convertFonts.sh;

# Install dockerize
ENV DOCKERIZE_VERSION v0.6.1
RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz

# Install Sqreen assets
RUN apt-get install -y debian-archive-keyring \
    && apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 7B5248ECE3E59766 \
    && UNIQUE_ID=`hostname -f` curl 'https://8dc0b36f0ea6f2f21b721765e10a7e02768cd1825b4551f4:@packagecloud.io/install/repositories/sqreen/sqreen/config_file.list?os=debian&dist=buster&name=${UNIQUE_ID}' > /etc/apt/sources.list.d/sqreen.list \
    && apt-get update -y \
    && apt-get install -y sqreen-agent sqreen-php-extension

# Clean-up
RUN apt-get clean \
    && apt-get remove -y gnupg \
    && apt-get autoremove -y \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Add a non-root user to prevent files being created with root permissions on host machine.
ARG PUID=1000
ARG PGID=1000
RUN groupadd -g $PGID audith && useradd -u $PUID -g audith -m audith
RUN usermod -aG sudo audith
RUN sed -i -e "s/%sudo.*/%sudo ALL=(ALL:ALL) NOPASSWD: ALL/g" /etc/sudoers

COPY ./profile /etc/profile
COPY ./profile /etc/bashrc

USER audith

# Update PATH
ENV PATH $PATH:/opt/woff2:/opt/css3_font_converter
