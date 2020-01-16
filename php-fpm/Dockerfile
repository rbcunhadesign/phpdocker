#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#
# To edit the 'php-fpm' base Image, visit its repository on Github
#    https://github.com/Laradock/php-fpm
#
# To change its version, see the available Tags on the Docker Hub:
#    https://hub.docker.com/r/laradock/php-fpm/tags/
#
# Note: Base Image name format {image-tag}-{php-version}
#

ARG LARADOCK_PHP_VERSION

FROM laradock/php-fpm:2.6.2-${LARADOCK_PHP_VERSION}

ARG LARADOCK_PHP_VERSION

# Set Environment Variables
ENV DEBIAN_FRONTEND noninteractive

# always run apt update when start and after add new source list, then clean up at end.
RUN set -xe; \
    apt-get update -yqq && \
    pecl channel-update pecl.php.net && \
    apt-get install -yqq \
      apt-utils \
      #
      #--------------------------------------------------------------------------
      # Mandatory Software's Installation
      #--------------------------------------------------------------------------
      #
      # Mandatory Software's such as ("mcrypt", "pdo_mysql", "libssl-dev", ....)
      # are installed on the base image 'laradock/php-fpm' image. If you want
      # to add more Software's or remove existing one, you need to edit the
      # base image (https://github.com/Laradock/php-fpm).
      #
      # next lines are here becase there is no auto build on dockerhub see https://github.com/laradock/laradock/pull/1903#issuecomment-463142846
      libzip-dev zip unzip \
      && docker-php-ext-configure zip \
      # Install the zip extension
      && docker-php-ext-install zip \
      && php -m | grep -q 'zip'

###########################################################################
# pgsql
###########################################################################

ARG INSTALL_PGSQL=false

RUN if [ ${INSTALL_PGSQL} = true ]; then \
    # Install the pgsql extension
    docker-php-ext-install pgsql \
;fi

###########################################################################
# xDebug:
###########################################################################

ARG INSTALL_XDEBUG=false

RUN if [ ${INSTALL_XDEBUG} = true ]; then \
  # Install the xdebug extension
  pecl install xdebug \
  && docker-php-ext-enable xdebug \
;fi
# Copy xdebug configuration for remote debugging
COPY ./xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

RUN sed -i "s/xdebug.remote_autostart=0/xdebug.remote_autostart=1/" /usr/local/etc/php/conf.d/xdebug.ini && \
    sed -i "s/xdebug.remote_enable=0/xdebug.remote_enable=1/" /usr/local/etc/php/conf.d/xdebug.ini && \
    sed -i "s/xdebug.cli_color=0/xdebug.cli_color=1/" /usr/local/etc/php/conf.d/xdebug.ini

###########################################################################
# PHP REDIS EXTENSION
###########################################################################

ARG INSTALL_PHPREDIS=false

RUN if [ ${INSTALL_PHPREDIS} = true ]; then \
    # Install Php Redis Extension
    pecl install -o -f redis \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable redis \
;fi

###########################################################################
# MongoDB:
###########################################################################

ARG INSTALL_MONGO=false

RUN if [ ${INSTALL_MONGO} = true ]; then \
    # Install the mongodb extension
    pecl install mongodb \
    && docker-php-ext-enable mongodb \
;fi

###########################################################################
# bcmath:
###########################################################################

ARG INSTALL_BCMATH=false

RUN if [ ${INSTALL_BCMATH} = true ]; then \
    # Install the bcmath extension
    docker-php-ext-install bcmath \
;fi

###########################################################################
# PHP OCI8:
###########################################################################

ARG INSTALL_OCI8=false

ENV LD_LIBRARY_PATH="/opt/oracle/instantclient_12_1"
ENV OCI_HOME="/opt/oracle/instantclient_12_1"
ENV OCI_LIB_DIR="/opt/oracle/instantclient_12_1"
ENV OCI_INCLUDE_DIR="/opt/oracle/instantclient_12_1/sdk/include"
ENV OCI_VERSION=12

RUN if [ ${INSTALL_OCI8} = true ]; then \
    # Install wget
    apt-get update && apt-get install --no-install-recommends -y wget \
    # Install Oracle Instantclient
    && mkdir /opt/oracle \
        && cd /opt/oracle \
        && wget https://github.com/diogomascarenha/oracle-instantclient/raw/master/instantclient-basic-linux.x64-12.1.0.2.0.zip \
        && wget https://github.com/diogomascarenha/oracle-instantclient/raw/master/instantclient-sdk-linux.x64-12.1.0.2.0.zip \
        && unzip /opt/oracle/instantclient-basic-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
        && unzip /opt/oracle/instantclient-sdk-linux.x64-12.1.0.2.0.zip -d /opt/oracle \
        && ln -s /opt/oracle/instantclient_12_1/libclntsh.so.12.1 /opt/oracle/instantclient_12_1/libclntsh.so \
        && ln -s /opt/oracle/instantclient_12_1/libclntshcore.so.12.1 /opt/oracle/instantclient_12_1/libclntshcore.so \
        && ln -s /opt/oracle/instantclient_12_1/libocci.so.12.1 /opt/oracle/instantclient_12_1/libocci.so \
        && rm -rf /opt/oracle/*.zip \
    # Install PHP extensions deps
    && apt-get update \
        && apt-get install --no-install-recommends -y \
            libaio-dev \
            freetds-dev && \
    # Install PHP extensions
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      echo 'instantclient,/opt/oracle/instantclient_12_1/' | pecl install oci8-2.0.10; \
    else \
      echo 'instantclient,/opt/oracle/instantclient_12_1/' | pecl install oci8; \
    fi \
        && docker-php-ext-configure pdo_oci --with-pdo-oci=instantclient,/opt/oracle/instantclient_12_1,12.1 \
        && docker-php-ext-configure pdo_dblib --with-libdir=/lib/x86_64-linux-gnu \
        && docker-php-ext-install \
                pdo_oci \
        && docker-php-ext-enable \
                oci8 \
  ;fi

###########################################################################
# Opcache:
###########################################################################

ARG INSTALL_OPCACHE=false

RUN if [ ${INSTALL_OPCACHE} = true ]; then \
    docker-php-ext-install opcache \
;fi

# Copy opcache configration
COPY ./opcache.ini /usr/local/etc/php/conf.d/opcache.ini

###########################################################################
# Human Language and Character Encoding Support:
###########################################################################

ARG INSTALL_INTL=false

RUN if [ ${INSTALL_INTL} = true ]; then \
    # Install intl and requirements
    apt-get install -y zlib1g-dev libicu-dev g++ && \
    docker-php-ext-configure intl && \
    docker-php-ext-install intl \
;fi

###########################################################################
# LDAP:
###########################################################################

ARG INSTALL_LDAP=false

RUN if [ ${INSTALL_LDAP} = true ]; then \
    apt-get install -y libldap2-dev && \
    docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ && \
    docker-php-ext-install ldap \
;fi

###########################################################################
# SQL SERVER:
###########################################################################

ARG INSTALL_MSSQL=false

RUN set -eux; \
  if [ ${INSTALL_MSSQL} = true ]; then \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      apt-get -y install freetds-dev libsybdb5 \
      && ln -s /usr/lib/x86_64-linux-gnu/libsybdb.so /usr/lib/libsybdb.so \
      && docker-php-ext-install mssql pdo_dblib \
      && php -m | grep -q 'mssql' \
      && php -m | grep -q 'pdo_dblib' \
    ;else \
      ###########################################################################
      # Ref from https://github.com/Microsoft/msphpsql/wiki/Dockerfile-for-adding-pdo_sqlsrv-and-sqlsrv-to-official-php-image
      ###########################################################################
      # Add Microsoft repo for Microsoft ODBC Driver 13 for Linux
      apt-get install -y apt-transport-https gnupg \
      && curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
      && curl https://packages.microsoft.com/config/debian/9/prod.list > /etc/apt/sources.list.d/mssql-release.list \
      && apt-get update -yqq \
      # Install Dependencies
      && ACCEPT_EULA=Y apt-get install -y unixodbc unixodbc-dev libgss3 odbcinst msodbcsql17 locales \
      && echo "en_US.UTF-8 UTF-8" > /etc/locale.gen \
      # link local aliases
      && ln -sfn /etc/locale.alias /usr/share/locale/locale.alias \
      && locale-gen \
      # Install pdo_sqlsrv and sqlsrv from PECL. Replace pdo_sqlsrv-4.1.8preview with preferred version.
      && if [ $(php -r "echo PHP_MINOR_VERSION;") = "0" ]; then \
        pecl install pdo_sqlsrv-5.3.0 sqlsrv-5.3.0 \
      ;else \
        pecl install pdo_sqlsrv sqlsrv \
      ;fi \
      && docker-php-ext-enable pdo_sqlsrv sqlsrv \
      && php -m | grep -q 'pdo_sqlsrv' \
      && php -m | grep -q 'sqlsrv' \
    ;fi \
  ;fi

###########################################################################
# ImageMagick:
###########################################################################

USER root

ARG INSTALL_IMAGEMAGICK=false

RUN if [ ${INSTALL_IMAGEMAGICK} = true ]; then \
    apt-get install -y libmagickwand-dev imagemagick \
    && pecl install imagick \
    && docker-php-ext-enable imagick \
;fi

###########################################################################
# YAML:
###########################################################################

USER root

ARG INSTALL_YAML=false

RUN if [ ${INSTALL_YAML} = true ]; then \
    apt-get install libyaml-dev -y \
    && pecl install yaml \
    && docker-php-ext-enable yaml \
;fi

###########################################################################
# Check PHP version:
###########################################################################

RUN set -xe; php -v | head -n 1 | grep -q "PHP ${LARADOCK_PHP_VERSION}."

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

COPY ./laravel.ini /usr/local/etc/php/conf.d
COPY ./xlaravel.pool.conf /usr/local/etc/php-fpm.d/

USER root

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

# Configure non-root user.
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

RUN groupmod -o -g ${PGID} www-data && \
    usermod -o -u ${PUID} -g www-data www-data

# Configure locale.
ARG LOCALE=POSIX
ENV LC_ALL ${LOCALE}

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000
