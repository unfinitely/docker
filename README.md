# Docker Image

[![php](https://github.com/unfinitely/docker/actions/workflows/php.yml/badge.svg)](https://github.com/unfinitely/docker/actions/workflows/php.yml) [![wp](https://github.com/unfinitely/docker/actions/workflows/wp.yml/badge.svg)](https://github.com/unfinitely/docker/actions/workflows/wp.yml) [![GitHub license](https://img.shields.io/github/license/unfinitely/docker)](https://github.com/unfinitely/docker/blob/main/LICENSE)

> A collection of [Docker](https://www.docker.com/) image for spinning-up local development environment.

## Image Stacks

Each of these Docker image provides the following stacks that (I think) would commonly be needed on development.

| Stack | `php` | `wp` |
| --- | --- | --- |
| Composer | ✅ | ✅ |
| Git | ✅ | ✅ |
| Subversion | - | ✅ |
| WP-CLI | - | ✅ |
| Mailhog | ✅ | ✅ |
| PHP `redis` | ✅ | ✅ |
| PHP `xdebug` | ✅ | ✅ |

## Usage

The image is available in [Github Package Registry](https://github.com/unfinitely/docker/pkgs/container/php), and you can run, for example, `docker pull ghcr.io/unfinitely/wp:8.0-apache` command to pull the image for WordPress that runs on PHP 8.0 with Apache or `docker pull ghcr.io/unfinitely/wp:8.0-fpm` to pull in the FPM variant that can be used with Nginx. Take a look at the [Wiki](https://github.com/unfinitely/docker/wiki) for a more real-world use cases of these images.
