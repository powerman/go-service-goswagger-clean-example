# Example Go service using go-swagger and The Clean Architecture
[![PkgGoDev](https://pkg.go.dev/badge/powerman/go-service-goswagger-clean-example)](https://pkg.go.dev/powerman/go-service-goswagger-clean-example) ![Go Report Card](https://goreportcard.com/badge/github.com/powerman/go-service-goswagger-clean-example)](https://goreportcard.com/report/github.com/powerman/go-service-goswagger-clean-example) [![CircleCI](https://circleci.com/gh/powerman/go-service-goswagger-clean-example.svg?style=svg)](https://circleci.com/gh/powerman/go-service-goswagger-clean-example) [![Coverage Status](https://coveralls.io/repos/github/powerman/go-service-goswagger-clean-example/badge.svg?branch=master)](https://coveralls.io/github/powerman/go-service-goswagger-clean-example?branch=master) [![Project Layout](https://img.shields.io/badge/Standard Go-Project Layout-informational)](https://github.com/golang-standards/project-layout) [![Release](https://img.shields.io/github/v/release/powerman/go-service-goswagger-clean-example)](https://github.com/powerman/go-service-goswagger-clean-example/releases/latest)

This project shows an example of how to use go-swagger accordingly to
Uncle Bob's "Clean Architecture".

Also it includes [go-swagger JSON Schema support
cheatsheet](docs/json-schema-cheatsheet.yml), which list all
validations/annotations for JSON body actually implemented by go-swagger
v0.18.0.

# Overview

## The Clean Architecture

[![The Clean Architecture](https://blog.cleancoder.com/uncle-bob/images/2012-08-13-the-clean-architecture/CleanArchitecture.jpg)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)

It's not a complete example of The Clean Architecture itself
(business-logic of this example is too trivial, so "Use Cases" layer in
package `app` embeds "Entities" layer), but it does show the most relevant
part: how to create "API Controller" layer in package `srv/openapi`
between code auto-generated by go-swagger and "Use Cases" layer in package
`app`. Also it includes "DB Gateway" layer in package `dal` (provided
trivial in-memory implementation is "DB" and "Gateway" layers at once).

## The hexagonal architecture, or ports and adapters architecture

It may be even easier to understand implemented architecture as "ports and
adapters":

- "ports" are defined as interfaces in `app/app.go` - they make it
  possible to easily test business-logic in `app` without any external
  dependencies by mocking all these interfaces.
- "adapters" are implemented in `srv/*` (serve project APIs), `dal`
  (access DB) and `svc/*` (use external services) packages - they can't be
  tested that easy, so they should be as thin and straightforward as
  possible and try hard to do nothing than convert ("adapt") data between
  format used by external world and our business-logic (package `app`).

## Structure of Go packages

- `api/*` - definitions of own and 3rd-party APIs/protocols and related
  auto-generated code
- `cmd/*` - main application(s)
- `internal/def` - defaults for both application(s) and tests
- `internal/config` - configuration(s) (default values, env, flags) for
  application(s) subcommands and tests
- `internal/app` - define interfaces ("ports") and implements business-logic
- `internal/srv/*`, `internal/dal`, `internal/svc/*` - implements
  "adapters" (for APIs/UI served by application(s), DB, access to external
  services)
- `pkg/*`, `internal/pkg/*` - helper packages, not related to architecture
  and business-logic (may be later moved to own modules and/or replaced by
  external dependencies)

## Features

- [X] Project structure (mostly) follow [Standard Go Project Layout](https://github.com/golang-standards/project-layout).
- [X] Strict but convenient golangci-lint configuration.
- [X] Easily testable code (thanks to The Clean Architecture).
- [X] Avoids (and resists to) using global objects (to make it possible to
  embed such microservices into modular monolith).
- [X] CLI subcommands support using [cobra](https://github.com/spf13/cobra).
- [X] Graceful shutdown support.
- [X] Configuration defaults can be overwritten by env vars and flags.
- [X] CORS support, so you can play with API using Swagger Editor tool.
- [X] Example go-swagger authentication and authorization.
- [X] Example tests, both unit and integration.
- [X] Production logging using [structlog](https://github.com/powerman/structlog).
- [X] Production metrics using Prometheus.
- [X] Docker and docker-compose support.

# Development

## Requirements

- Go 1.15
- [Docker](https://docs.docker.com/install/) 19.03+
- [Docker Compose](https://docs.docker.com/compose/install/) 1.25+
- Tools used to build/test project:

```sh
curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOPATH)/bin v1.30.0
go get gotest.tools/gotestsum@v0.5.3
curl -sSfL https://github.com/go-swagger/go-swagger/releases/download/v0.25.0/swagger_$(uname)_amd64 | install /dev/stdin $(go env GOPATH)/bin/swagger
go get github.com/golang/mock/mockgen@v1.4.4
go get github.com/cheekybits/genny@master
```

## Setup

1. After cloning the repo copy `env.sh.dist` to `env.sh`.
2. Review `env.sh` and update for your system as needed.
3. It's recommended to add shell alias `alias dc="if test -f env.sh; then
   source env.sh; fi && docker-compose"` and then run `dc` instead of
   `docker-compose` - this way you won't have to run `source env.sh` after
   changing it.

## Usage

To develop this project you'll need only standard tools: `go generate`,
`go test`, `go build`, `docker build`. Provided scripts are for
convenience only.

- Always load `env.sh` *in every terminal* used to run any project-related
  commands (including `go test`): `source env.sh`.
    - When `env.sh.dist` change (e.g. by `git pull`) next run of `source
      env.sh` will fail and remind you to manually update `env.sh` to
      match current `env.sh.dist`.
- `go generate ./...` - do not forget to run after making changes related
  to auto-generated code
- `go test ./...` - test project (excluding integration tests), fast
- `./scripts/test` - thoroughly test project, slow
- `./scripts/build` - build docker image and binaries in `bin/`
    - Then use mentioned above `dc` (or `docker-compose`) to run and
      control the project.
    - Access project at host/port(s) defined in `env.sh`.

### Cheatsheet
```sh
dc up -d --remove-orphans               # (re)start all project's services
dc logs -f -t                           # view logs of all services
dc logs -f SERVICENAME                  # view logs of some service
dc ps                                   # status of all services
dc restart SERVICENAME
dc exec SERVICENAME COMMAND             # run command in given container
dc stop && dc rm -f                     # stop the project
docker volume rm PROJECT_SERVICENAME    # remove some service's data
```

It's recommended to avoid `docker-compose down` - this command will also
remove docker's network for the project, and next `dc up -d` will create a
new network… repeat this many enough times and docker will exhaust
available networks, then you'll have to restart docker service or reboot.

# Run

Use of the `./scripts/build` script is optional (it's main feature is
embedding git version into compiled binary), you can use usual
`go get|install|build` to get the application instead.

```
$ ./scripts/build
$ ./bin/address-book -h
Example microservice with OpenAPI

Usage:
  address-book [flags]
  address-book [command]

Available Commands:
  help        Help about any command
  serve       Starts microservice

Flags:
  -h, --help                    help for address-book
      --log.level OneOfString   log level [debug|info|warn|err] (default debug)
  -v, --version                 version for address-book

Use "address-book [command] --help" for more information about a command.

$ ./bin/address-book serve -h
Starts microservice

Usage:
  address-book serve [flags]

Flags:
  -h, --help                        help for serve
      --host NotEmptyString         host to serve OpenAPI (default localhost)
      --metrics.port Port           port to serve Prometheus metrics (default 9000)
      --port Port                   port to serve OpenAPI (default 8000)
      --timeout.shutdown Duration   must be less than 10s used by 'docker stop' between SIGTERM and SIGKILL (default 9s)
      --timeout.startup Duration    must be less than swarm's deploy.update_config.monitor (default 3s)

Global Flags:
      --log.level OneOfString   log level [debug|info|warn|err] (default debug)

$ ./bin/address-book -v
address-book version v1.0.0 5e45f44 2020-09-03_15:15:53 go1.15.1

$ ./bin/address-book serve
 address-book: inf      main: `started` version v1.0.0 5e45f44 2020-09-03_15:15:53
 address-book: inf   openapi: `OpenAPI protocol` version 0.2.0
 address-book: inf     serve: `serve` localhost:9000 [Prometheus metrics]
 address-book: inf     serve: `serve` 127.0.0.1:8000 [OpenAPI]
 address-book: inf   swagger: `Serving address book at http://127.0.0.1:8000`
 address-book: dbg   openapi: 127.0.0.1:36500           POST    contacts: `calling AddContact` admin
 address-book: dbg       dal: 127.0.0.1:36500           POST    contacts: `contact added` admin
 address-book: inf   openapi: 127.0.0.1:36500       201 POST    contacts: `handled` admin
 address-book: dbg   openapi: 127.0.0.1:36502           POST    contacts: `calling AddContact` admin
 address-book: dbg       dal: 127.0.0.1:36502           POST    contacts: `contact added` admin
 address-book: inf   openapi: 127.0.0.1:36502       201 POST    contacts: `handled` admin
 address-book: inf   openapi: 127.0.0.1:36504       200 GET     contacts: `handled` admin
 address-book: inf   openapi: 127.0.0.1:36508       200 GET     contacts: `handled` user
 address-book: inf   openapi: 127.0.0.1:36510       401 GET     contacts: `handled`
 address-book: inf   openapi: 127.0.0.1:36518       403 POST    contacts: `handled`
^C
 address-book: inf   swagger: `Shutting down... `
 address-book: inf   swagger: `HTTP server Shutdown: context deadline exceeded`
 address-book: inf   swagger: `Stopped serving address book at http://127.0.0.1:8000`
 address-book: inf     serve: `shutdown` [OpenAPI]
 address-book: inf     serve: `shutdown` [Prometheus metrics]
 address-book: inf      main: `finished` version v1.0.0 5e45f44 2020-09-03_15:15:53
```

# TODO

- [ ] Add CI/CD using GitHub Actions.
- [ ] Update JSON Schema support cheatsheet to latest go-swagger version.
- [ ] Replace trivial in-memory DAL with more complete one based on
  Postgresql with metrics and migrations support.
- [ ] Add cookie-based auth with CSRF middleware.
- [ ] Add an example of adapter for external service in `svc/something`.
