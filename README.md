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

- The [jq](https://stedolan.github.io/jq/) command, a CLI JSON processor.

## Pre/Post commands

Some packages may require additional setup steps: setting up a web server to test an HTTP client, seeding a database or cache service, etc.
Other times, you may want to do additional reporting, particularly if the QA command failed.

To enable this, you may create one or both of the following files in your package:

- `.laminas-ci/pre-run.sh`
- `.laminas-ci/post-run.sh`

(Note: the files MUST be executable to be consumed!)

These run immediately before and after the QA command, respectively.
The `.laminas-ci/pre-run.sh` command will receive the following arguments:

- `$1`: the user the QA command will run under
- `$2`: the WORKDIR path
- `$3`: the `$JOB` passed to the entrypoint (see above)

The `.laminas-ci/post-run.sh` command will receive these arguments:

- `$1`: the exit status of the QA command
- `$2`: the user the QA command will run under
- `$3`: the WORKDIR path
- `$4`: the `$JOB` passed to the entrypoint (see above)

### Parsing the $JOB

You may want to grab elements of the `$JOB` argument in order to branch logic.
Generally speaking, you can use the [jq](https://stedolan.github.io/jq/) command to get at this data.
As an example, to get the PHP version:

```bash
JOB=$3
PHP_VERSION=$(echo "${JOB}" | jq -r '.php')
```

If you want to conditionally skip setup based on the command (in this case, exiting early if the command to run is not phpunit):

```bash
JOB=$3
COMMAND=$(echo "${JOB}" | jq -r '.command')
if [[ ! ${COMMAND} =~ phpunit ]];then
    exit 0
fi
```

Perhaps after running a job against locked dependencies, you want to see if newer versions are available:

```bash
JOB=$3
DEPS=$(echo "${JOB}" | jq -r '.dependencies')
if [[ "${DEPS}" != "locked" ]];then
    exit 0
fi
# check for newer versions...
```

If you need access to the list of extensions or php.ini directives, you should likely write a script in PHP or node to do so.

## Using the container locally to run tests

The [standard Laminas Continuous Integration workflow](https://gist.github.com/weierophinney/9decd19f76b7d9745c6559074053fa65) defines one job using the laminas-ci-matrix-action to create the matrix, and defines another job to run the various jobs in the matrix that consumes it.
Unfortunately, as of this writing, tools like [nektos/act](https://github.com/nektos/act) are unable to work with job/step dependencies, nor with workflow metadata expressions, meaning you cannot run the full suite at once.

What you _can_ do, however, is run individual jobs.

First, pull the container locally:

```bash
$ docker pull ghcr.io/laminas/laminas-continuous-integration-container:1
```

Once you have pulled it, you can run individual jobs.
The tricks to remember are:

- You need to set bind the package directory as a volume.
- You need to set the container WORKDIR to that volume.
- You need to provide the job JSON.

As an example, if you wanted to run the CS checks under PHP 7.4 using locked dependencies, you could do something like the following:

```bash
$ docker run -v $(realpath .):/github/workspace -w=/github/workspace laminas-check-runner:latest '{"php":"7.4","deps":"locked","extensions":[],"ini":["memory_limit=-1"],"command":"./vendor/bin/phpcs"}'
```

The trick to remember: the job JSON should generally be in single quotes, to allow the `"` characters used to delimit properties and strings in the JSON to not cause interpolation issues.

## Tags

- ghcr.io/laminas/laminas-continuous-integration-container:1 (latest v1 release)
- ghcr.io/laminas/laminas-continuous-integration-container:1.1 (latest v1.1.x release)
- ghcr.io/laminas/laminas-continuous-integration-container:1.1.0 (v1.1.0 release)
