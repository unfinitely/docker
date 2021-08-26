FROM wordpress:php8.0-fpm

LABEL org.opencontainers.image.source https://github.com/unfinitely/docker
LABEL org.opencontainers.image.vendor Unfinitely
LABEL org.opencontainers.image.description "Image to run WordPress development environment"

# Install extra packages
RUN set -ex; \
	apt-get update \
	&& EXTRA_PACKAGES=" \
    ca-certificates \
		subversion \
		default-mysql-client \
		unzip \
		git \
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

# Install WP-CLI
ENV WP_CLI_VERSION 2.5.0
ENV WP_CLI_SHA512 08dd9035fda1d529807380d5b757839e2809e289eb1a698fe33e7e21a1431d3f77c551c2b2db5adc55083d5075ea4137407994111890f765e790a97e6d9ca7af

RUN set -ex; \
	curl -o /usr/local/bin/wp.phar -fSL "https://github.com/wp-cli/wp-cli/releases/download/v${WP_CLI_VERSION}/wp-cli-${WP_CLI_VERSION}.phar" \
	&& echo "$WP_CLI_SHA512 */usr/local/bin/wp.phar" | sha512sum -c - \
	&& chmod +x /usr/local/bin/wp.phar \
	&& mv /usr/local/bin/wp.phar /usr/local/bin/wp \
	&& wp cli info

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
RUN set -ex; \
    sed -i 's/^\(error_reporting\s*=\s*\).*\$/\1E_ALL/' /usr/local/etc/php/conf.d/error-logging.ini; \
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
      echo "alias wp='runuser -u www-data -- $(which wp)'"; \
    } >> ~/.bashrc;
