FROM phoenixleonardo/php-55-fpm-alpine
MAINTAINER Leonardo <phoenixleonardo@gmail.com>
# RUN docker-php-ext-install mysqli pdo pdo_mysql && docker-php-ext-enable pdo_mysql

RUN apk add --no-cache --virtual .build-deps \
            # for all
            git \
            #for bzip
            bzip2-dev \
            #for curl
            curl-dev \
            # for intl extension
            gettext-dev \
            zlib-dev \
            fcgi \
            # for imagick
            autoconf \
            imagemagick-dev\
            libtool\
            # dev deps for gd
            freetype-dev \
            libjpeg-turbo-dev \
            libpng-dev \
            # for intl
            icu-dev \
            #for imap
            imap-dev \
            #for mysql
            mariadb-dev \
            # for mcrypt
            libmcrypt-dev \
            # for memcache
            libmemcached-dev \
            # for soap
            libxml2-dev \
            # for xsl
            libxslt-dev \
            # for ldap
            openldap-dev \
            # to build xdebug from PECL
            $PHPIZE_DEPS \

    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
     && docker-php-ext-install \
         bcmath \
         bz2 \
         calendar \
         curl \
         exif \
         fileinfo \
         ftp \
         gd \
         gettext \
         imap \
         intl \
         ldap \
         mbstring \
         mcrypt \
         mysql \
         mysqli \
         opcache \
         pdo \
         pdo_mysql \
         soap \
         sysvsem \
         sysvshm \
         xmlrpc \
         xsl \
         zip \

         && git clone --branch 1.x https://github.com/twigphp/Twig.git /tmp/twig \
        && ( \
                cd /tmp/twig/ext/twig \
                && phpize \
                && ./configure \
                && make -j$(nproc) \
                && make install \
                && cd .. \
        ) \

        && rm -r /tmp/twig \

#####   Using Xdebug 2.5.5 as support for PHP 5.5 has been dropped in newer versions   #####
        && pecl install xdebug-2.5.5 \
        && pecl install xhprof-beta \
        && pecl install uploadprogress \
        && printf "\n" | pecl install imagick \
        && pecl install mongodb \
        && pecl install memcache \
        && docker-php-ext-enable opcache \
        xdebug \
        xhprof \
        imagick \
        mongodb \
        memcache \
        twig \
        uploadprogress \

        &&  echo -e "\n opcache.enable=1 \n opcache.enable_cli=1 \n opcache.memory_consumption=128 \n opcache.interned_strings_buffer=8 \n opcache.max_accelerated_files=4000 \n opcache.revalidate_freq=60 \n opcache.fast_shutdown=1" >> /usr/local/etc/php/conf.d/docker-php-ext-opcache.ini \
        &&  echo -e "\n xdebug.remote_enable=1 \n xdebug.remote_host=localhost \n xdebug.remote_port=9000" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
        &&  echo -e "\n xhprof.output_dir='/var/tmp/xhprof'" >> /usr/local/etc/php/conf.d/docker-php-ext-xhprof.ini \

        # && pecl install intl\
        # && docker-php-ext-enable intl \
     
    # next will add runtime deps for php extensions
    # what this does is checks the necessary packages for php extensions Shared Objects
    # and adds those (won't be removed when .build-deps are)
    # && docker-php-ext-enable \
        #  xdebug\
     #    uploadprogress\
       
    && runDeps="$( \
            scanelf --needed --nobanner --recursive \
                /usr/local/lib/php/extensions \
                | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
                | sort -u \
                | xargs -r apk info --installed \
                | sort -u \
        )" \
    && apk add --virtual .phpexts-rundeps $runDeps \
    && apk del .build-deps

# modify www-data user to have id 1000
RUN apk add \
    --no-cache \
    --repository http://dl-3.alpinelinux.org/alpine/edge/community/ --allow-untrusted \
    --virtual .shadow-deps \
    shadow \
    && usermod -u 1000 www-data \
    && groupmod -g 1000 www-data \
    && apk del .shadow-deps \
# Install composer
    && php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php -r "copy('https://composer.github.io/installer.sig', 'signature');" \
    && php -r "if (hash_file('SHA384', 'composer-setup.php') === trim(file_get_contents('signature'))) { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
    && php -r "unlink('composer-setup.php');"