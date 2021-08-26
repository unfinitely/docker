FROM php:7.4-fpm

LABEL org.opencontainers.image.source https://github.com/unfinitely/docker
LABEL org.opencontainers.image.vendor Unfinitely
LABEL org.opencontainers.image.description "Image to run development environment with PHP"

# Install extra packages
RUN set -ex; \
	apt-get update \
	&& EXTRA_PACKAGES=" \
		default-mysql-client \
		git \
		unzip \
    ca-certificates \
	" \
	&& apt-get install -y $EXTRA_PACKAGES --no-install-recommends \
	&& apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /var/cache/*

# Install additional PHP extensions
ENV REDIS_VERSION 5.3.4
ENV XDEBUG_VERSION 3.0.4

RUN set -ex; \
    pecl install "redis-${REDIS_VERSION}"; \
    pecl install "xdebug-${XDEBUG_VERSION}"; \
    docker-php-ext-enable redis xdebug \
    && rm -rf /var/lib/apt/lists/*

# Install Composer
ENV COMPOSER_VERSION 2.1.3

RUN set -eux; \
  curl \
		--silent \
		--fail \
		--location \
		--retry 3 \
		--output /tmp/installer.php \
		--url https://raw.githubusercontent.com/composer/getcomposer.org/cb19f2aa3aeaa2006c0cd69a7ef011eb31463067/web/installer; \
  php -r " \
    \$signature = '48e3236262b34d30969dca3c37281b3b4bbe3221bda826ac6a9a62d6444cdb0dcd0615698a5cbe587c3f0fe57a54d8f5'; \
    \$hash = hash('sha384', file_get_contents('/tmp/installer.php')); \
    if (!hash_equals(\$signature, \$hash)) { \
      echo 'Integrity check failed, installer is either corrupt or worse.' . PHP_EOL; \
      exit(1); \
    }" \
  ; \
  php /tmp/installer.php --no-ansi --install-dir=/usr/local/bin --filename=composer --version=${COMPOSER_VERSION}; \
  composer --ansi --version --no-interaction; \
	composer diagnose; \
	rm -f /tmp/installer.php;

# Install MailHog sendmail client
ENV MHSENDMAIL_VERSION 0.2.0

RUN set -ex; \
    ARCH= && dpkgArch="$(dpkg --print-architecture)" && \
    case "${dpkgArch##*-}" in \
      i386) ARCH='386';; \
      amd64) ARCH='amd64';; \
      arm64|armhf|armel) ARCH='arm';; \
      *) echo "Unsupported architecture: ${dpkgArch}"; exit 1;; \
    esac; \
    curl -o "/tmp/mhsendmail" \
    -fSL "https://github.com/mailhog/mhsendmail/releases/download/v${MHSENDMAIL_VERSION}/mhsendmail_linux_${ARCH}" \
    && chmod +x /tmp/mhsendmail \
    && mv /tmp/mhsendmail /usr/local/bin/mhsendmail \
    && rm -f /tmp/mhsendmail

# PHP Configurations

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN set -eux; \
	docker-php-ext-enable opcache; \
	{ \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=2'; \
		echo 'opcache.fast_shutdown=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini

# https://www.php.net/manual/en/errorfunc.constants.php
RUN { \
		echo 'error_reporting = E_ALL'; \
		echo 'display_errors = Off'; \
		echo 'display_startup_errors = Off'; \
		echo 'log_errors = On'; \
		echo 'error_log = /dev/stderr'; \
		echo 'log_errors_max_len = 1024'; \
		echo 'ignore_repeated_errors = On'; \
		echo 'ignore_repeated_source = Off'; \
		echo 'html_errors = Off'; \
	} > /usr/local/etc/php/conf.d/error-logging.ini

RUN set -ex; \
# Basic Xdebug configuration aim for debugging.
    { \
      echo 'xdebug.mode=debug'; \
      echo 'xdebug.client_host=host.docker.internal'; \
	  } >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Add some handy commands alias
#
# Alias only works for interactive/logged-in shell (Bash) session.
# Running the full command would still be needed in
# non-interactive shell session.
# Add some handy commands alias
RUN set -ex; \
    sed -i '/^# export LS_OPTIONS=/s/^#//; /^# alias /s/^#//' ~/.bashrc; \
    { \
      echo "alias lt='ls $LS_OPTIONS --human-readable --size -1 -S --classify'"; \
    } >> ~/.bashrc;
