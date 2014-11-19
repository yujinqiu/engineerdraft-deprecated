---
title: "30 days of hacking Docker"
description: "Post about developing Docker and Go in common."
date: "2014-06-11"
categories:
    - "go"
    - "docker"
---

### Prelude

Yesterday I finished my first 30-day [streak](https://github.com/LK4D4) on GitHub.
Most of contributions were to [Docker](https://github.com/dotcloud/docker) --
the biggest opensource project on Go. I learned a lot in this month, and it was
really cool. I think that this is mostly because of Go language. I've been
programming on Python for five years and I was never so excited about open source,
because Python is not even half so fun as Go.

### 1. Tools

There are a lot of tools for go, some of them just are "must have".

[Goimports](http://godoc.org/code.google.com/p/go.tools/cmd/goimports) - like
`go fmt` but with cool imports handling, I really think that `go fmt` needs to
be replaced with `Goimports` in future Go versions.

[Vet](http://godoc.org/code.google.com/p/go.tools/cmd/vet) - analyzes code for
some suspicious constructs. You can find with it: bad format strings, unreachable
code, passing mutex by value and etc.
[PR about vet erros in Docker](https://github.com/dotcloud/docker/pull/6269).

[Golint](https://github.com/golang/lint) - checks code for
[google style guide](https://code.google.com/p/go-wiki/wiki/CodeReviewComments).


### 2. Editor

I love my awesome vim with awesome [vim-go](https://github.com/fatih/vim-go) plugin,
which is integrated with tools mentioned above.
It formats code for me, adds needed imports, removes unused imports, shows
documentation, supports tagbar and more. And my favourite - go to definition. I
really suffered without it :) With vim-go my development rate became faster
than I could imagine. You can see my config in my dotfiles
[repo](https://github.com/LK4D4/dotfiles).


### 3. Race detector

This is one of the most important and one of the most underestimated thing.
Very useful and very easy to use. You can find description and examples
[here](http://blog.golang.org/race-detector). I've found many race conditions
with this tool ([#1](https://github.com/dotcloud/docker/pull/6118),
[#2](https://github.com/dotcloud/docker/pull/6150),
[#3](https://github.com/dotcloud/docker/pull/6214),
[#4](https://github.com/dotcloud/docker/pull/6232),
[#5](https://github.com/gorilla/context/pull/14)).


### 4. Docker specific

Docker has very smart and friendly community. You can always ask for help about
hacking in #docker-dev on Freenode. But I'll describe some simple tasks that appears
when you try to hack docker first time.

#### Tests
There are three kinds of tests in docker repo:

* `unit` - unit tests(ah, we all know what unit tests are, right?). These tests
spreaded all over repository and can be run by `make test-unit`. You can run
tests for one directory, specifying it in `TESTDIRS` variable. For example

    ```
    TESTDIRS="daemon" make test-unit
    ```

    will run tests only for daemon directory.

* `integration-cli` - integration tests, that use external docker commands
(for example `docker build`, `docker run`, etc.). It is very easy to write this
kind of tests and you should do it if you think that your changes can change
Docker's behavior from client's point of view. These tests are located in `integration-cli`
directory and can be run by `make test-integration-cli`. You can run one or more
specific tests with setting `TESTFLAGS` variable. For example

    ```
    TESTFLAGS="-run TestBuild" make test-integration-cli
    ```

    will run all tests whose names starts with `TestBuild`.

* `integration` - integration tests, that use internal docker datastructures.
It is deprecated now, so if you want to write tests you should prefer
`integration-cli` or `unit`. These tests are located in `integration` directory and
can be run by `make test-integration`.

All tests can be run by `make test`.

#### Build and run tests on host
All `make` commands execute in docker container, it can be pretty annoying to
build container just for running unit tests for example.

So, for running unit test on host machine you need canonical Go
[workspace](http://golang.org/doc/code.html#Workspaces). When it's ready you can
just do symlink to docker repo in `src/github.com/dotcloud/docker`. But we still
need right `$GOPATH`, here is the trick:

    export GOPATH=<workspace>/src/github.com/dotcloud/docker/vendor:<workspace>

And then, for example you can run:

    go test github.com/dotcloud/docker/daemon/networkdriver/ipallocator

Some tests require external libs for example `libdevmapper`, you can disable
it with `DOCKER_BUILDTAGS` environment variable. For example:

    export DOCKER_BUILDTAGS='exclude_graphdriver_devicemapper exclude_graphdriver_aufs'

For fast building dynamic binary you can use this snippet in docker repo:

    export AUTO_GOPATH=1
    export DOCKER_BUILDTAGS='exclude_graphdriver_devicemapper exclude_graphdriver_aufs'
    hack/make.sh dynbinary

I use that `DOCKER_BUILDTAGS` for my `btrfs` system, so if you use `aufs` or
`devicemapper` you should change it for your driver.

#### Race detection
To enable race detection in docker I'm using patch:

    diff --git a/hack/make/binary b/hack/make/binary
    index b97069a..74b202d 100755
    --- a/hack/make/binary
    +++ b/hack/make/binary
    @@ -6,6 +6,7 @@ DEST=$1
     go build \
            -o "$DEST/docker-$VERSION" \
            "${BUILDFLAGS[@]}" \
    +       -race \
            -ldflags "
                    $LDFLAGS
                    $LDFLAGS_STATIC_DOCKER

After that all binaries will be with race detection. Note that this will slow
docker a lot.

#### Docker-stress
There is amazing
[docker-stress](https://github.com/spotify/docker-stress) from Spotify for
Docker load testing. Usage is pretty straightforward:

    ./docker-stress -c 50 -t 5

Here 50 clients are trying to run containers, which will alive for five seconds.
`docker-stress` uses only `docker run` jobs for testing, so I prefer also to run in
parallel sort of:

    docker events
    while true; do docker inspect $(docker ps -lq); done
    while true; do docker build -t test test; done

and so on.

#### Useful links
You definitely need to read
[Contributing to Docker](https://github.com/dotcloud/docker/blob/master/CONTRIBUTING.md)
and [Setting Up a Dev Environment](https://github.com/dotcloud/docker/blob/master/docs/sources/contributing/devenvironment.md).
I really don't think that something else is needed for Docker hacking start.


### Conclusion

This is all that I wanted to tell you about my first big opensource experience.
Also, just today Docker folks launched some
[new projects](https://github.com/docker) and I am very excited about it.
So, I want to invite you all to the magical world of Go, Opensource and,
of course, Docker.
