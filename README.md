# Laminas continuous integrations base container

This repository provides the base container image for use with the [laminas/laminas-continuous-integration-action](https://github.com/laminas/laminas-continuous-integration-action) GitHub Action.

It builds off the ubuntu:focal image, installs the [Sury PHP repository](https://deb.sury.org/), and installs PHP versions 5.6, 7.0, 7.1, 7.2, 7.3, 7.4, and 8.0, each with the following extensions:

- bz2
- curl
- fileinfo
- intl
- json
- mbstring
- phar
- readline
- sockets
- xml
- xsl
- zip

It defines an entrypoint that accepts a single argument, a JSON string. The JSON string should contain the following elements:

- command: (string; required) the command to run (e.g., `./vendor/bin/phpunit`)
- php: (string; required) the PHP version to use when running the check
- extensions: (array of strings; optional) additional extensions to install.
  The names used should correspond to package names from the Sury repository, minus the `php{version}-` prefix.
  As examples, "sqlite3" or "tidy".
- ini: (array of strings; optional) php.ini directives to use.
  Each item should be the full directive; e.g., `memory_limit=-1` or `date.timezone=America/New_York`.
- dependencies: (string; optional) the dependency set to run against: lowest, locked, or latest.
  If not provided, "locked" is used.

## Other tools available

The container provides the following tools:

- Composer (v2 release)

- [cs2pr](https://github.com/staabm/annotate-pull-request-from-checkstyle), which creates PR annotations from checkstyle output. If a tool you are using, such as `phpcs`, provides checkstyle output, you can pipe it to `cs2pr` to create PR annotations from errors/warnings/etc. raised.

- A `markdownlint` binary, via the [DavidAnson/markdownlint-cli2](https://github.com/DavidAnson/markdownlint-cli2) package.
  A default configuration is provided that disables the following rules:

  - MD013 (line-length)
  - MD014 (dollar signs used before commands without showing output)
  - MD024 (duplicate header)
  - MD028 (blank line inside block quote)
  - MD034 (bare URLs)

  Consumers can provide their own rules via a [.markdownlint.json](https://github.com/DavidAnson/markdownlint-cli2#markdownlintjsonc-or-markdownlintjson) file.

- A `yamllint` binary, via the [adrienverge/yamllint](https://github.com/adrienverge/yamllint) package.


## Tags

- ghcr.io/laminas/laminas-continuous-integration-container:1 (latest v1 release)
- ghcr.io/laminas/laminas-continuous-integration-container:1.0 (latest v1.0.x release)
- ghcr.io/laminas/laminas-continuous-integration-container:1.0.0 (v1.0.0 release)
