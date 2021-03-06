# vgo

![Go_Logo](/images/Go_Logo_Aqua.svg)

Getting familiar with [vgo](http://github.com/golang/vgo/), the latest Golang package manager.

[In the go community `vgo` is also known as [go modules](http://github.com/golang/go/wiki/Modules).
 Older package managers include [dep](http://golang.github.io/dep/) and [glide](http://glide.sh).]

Probably the place to start is with Russ Cox's blog:

    http://research.swtch.com/vgo-tour

[UPDATE: `vgo` was released with Go 1.11 while Russ Cox's post refers to an earlier implementation.
There are substantial differences, so Russ Cox's blog should only be read for an overview
and not considered to be authoritative. As well, `vgo` does not seem to follow standard \*nix
conventions for command-line tools (perhaps this will change in the future).]

## Contents

- [Rationale](#rationale)
- [Requirements](#requirements)
- [To Install](#to-install)
- [To Remove](#to-remove)
- [To Use](#to-use)
- [Git](#git)
- [Caches](#caches)
    - [Build cache](#build-cache)
    - [Module cache](#module-cache)
- [Dependencies](#dependencies)
    - [Vendoring Dependencies](#vendoring-dependencies)
    - [Updating Dependencies](#updating-dependencies)
- [GO111MODULE](#go111module)
- [My recommended workflow](#my-recommended-workflow)
    - [Step 1](#step-1)
    - [Step 2](#step-2)
    - [Step 3](#step-3)
- [go mod edit](#go-mod-edit)
- [Travis CI](#travis-ci)
- [Dependency Scanning](#dependency-scanning)
- [Cloud Functions](#cloud-functions)
- [Reference](#reference)
- [To Do](#to-do)

## Rationale

Dependency management in Golang has always been tricky. The original developers flagged
it as TBD and it's been problematic ever since. Building upon successful package managers
such as `Composer`, `npm`, and `pip` the Golang team came up with `vgo` - which uses a lot
of best practices.

Not everyone is happy about it though; for a useful recap of package management in Go
the following blog post is worth a read:

    http://codeengineered.com/blog/2018/golang-godep-to-vgo/

> vgo was built when the Go lead at Google went off on his own to build a solution.

Sam Boyer and the `dep` team do not sound happy either:

    https://golang.github.io/dep/blog/2018/07/25/announce-v0.5.0.html

> That means there's no choosing between "vgo/modules or dep." It'll be "vgo, or another language."

As `dep` is still being actively maintained, I think if I was using `dep` I would stick
with it until there was a compelling reason to switch to `vgo`.

My personal feeling is that - while I understand the attraction of a central dependency
store - dependencies should be bundled with the code that depends on them (links can go
dead and sometimes locating dependencies can be a trial). So I am not a fan of caching
downloaded modules centrally - or even caching build results centrally. I like to be
in control and am not a fan of having to locate hidden caches just so that I can nuke
them in order to get a 'clean' build. I would prefer that this should be under developer
control - and more transparent.

[This is a use case for `go mod vendor`, see [Vendoring Dependencies](#vendoring-dependencies)
 below for more details.]

I will use the `main.go` file from my [UI repo](http://github.com/mramshaw/ui) for testing.

[The dependencies for this are tricky so it is a good initial test.]

## Requirements

* a recent version of Golang (preferably a released version such as 1.11 or greater)

## To Install

The command to run to install `vgo`:

    $ go get -u golang.org/x/vgo

[UPDATE: `vgo` was the pre-release version - with 1.11 the new behaviour is activated
 by specifying `GO111MODULE=on` prior to the `go` command - as can be seen below.
 Presumably with later releases this will become the default option.]

## To Remove

As `vgo` is no longer required with Golang 1.11 (and later) it may be removed.

The command to run to remove `vgo`:

    $ go clean -i golang.org/x/vgo

This will leave some cruft in the various Go build and test caches, which
seems to be consistent with most of the other package managers (such as
`npm` et al). In practice, the normal procedure seems to be to either delete
the caches when disk space is needed or else to virtualize the build process
(by using either `docker` or `vagrant`).

For a dry run of the above command:

    $ go clean -n -i golang.org/x/vgo

As noted above, this will still leave some remaining cruft which may be cleaned
up manually.

UPDATE: It seems that specifying `GO111MODULE=on` installs `vgo` as a dependency,
which feels like a bit of a bodge. Perhaps this will be eliminated with later
versions of Go.

Presumably the build and test caches can be cleaned out as follows:

    $ go clean -cache -testcache -modcache

[Add an `-x` option to see the remove commands as they are executed.]

At least with `npm` it is possible to define build and runtime dependencies.

It would be nice if `vgo` had this level of flexibility.

## To Use

The command to run to build the project:

    $ GO111MODULE=on go build

This builds a binary called `vgo` (not really what we want), so:

    $ GO111MODULE=on go build -o greeting

We can execute this as follows:

    $ ./greeting

And everything works as expected. Huzzah!

## Git

The build generates `go.mod` and `go.sum` files. These should be checked into the Git repo.

Or perhaps not ... Dave Cheney has some interesting things to say:

    http://dave.cheney.net/2018/07/14/taking-go-modules-for-a-spin

And:

    http://dave.cheney.net/2018/07/16/using-go-modules-with-travis-ci

[This last post is pretty interesting as it focuses on Travis integration. By default
Go will continue to use pre-1.11 build behaviour unless `GO111MODULE=on` is specified.
So it seems that modules are a build-breaking issue. I wonder how this squares with the
[Go 1 Compatibility idea](https://golang.org/doc/go1compat)?]

## Caches

Dave Cheney's blog post has a useful note from Russ Cox about caches.

Recent versions of Go have introduced caches, both of __test__ results and __build__ results.

These probably correspond to the `-testcache` and `-cache` options for `go clean`.

The new cache seems to correspond with the `-modcache` option for `go clean`.

#### Build cache

> The build cache ($GOCACHE, defaulting to $HOME/.cache/go-build) is for storing recent compilation results

Lets have a look at it:

```bash
$ cat ~/.cache/go-build/README
This directory holds cached build artifacts from the Go build system.
Run "go clean -cache" if the directory is getting too large.
See golang.org to learn more about Go.
$
```

#### Module cache

> The module cache ($GOPATH/src/mod, defaulting to $HOME/go/src/mod) is for storing downloaded source code

This folder does not exist on my computer, I suspect it is actually the following:

```bash
$ ls -al ~/go/pkg/mod/cache
total 16
drwxrwxr-x 4 owner owner 4096 Oct 12 11:36 .
drwxrwxr-x 6 owner owner 4096 Oct 13 15:48 ..
drwxrwxr-x 5 owner owner 4096 Oct 13 15:48 download
drwxrwxr-x 9 owner owner 4096 Oct 13 15:48 vcs
$
```

## Dependencies

The following command will create `go.mod` and `go.sum` files:

    $ GO111MODULE=on go build -o greeting

However, under some circumstances it may be necessary to first specify the package:

    $ go mod init github.com/mramshaw/vgo

[This will create a `go.mod` file - the `go.sum` file will be created by a build.]

#### Vendoring Dependencies

The `go mod vendor` command will populate the required dependencies.

```bash
$ go mod vendor
go: finding github.com/andlabs/ui latest
$
```

[It may be first necessary to specify the package (as above) via `go mod init`.]

Some useful help:

```bash
$ go mod help vendor
usage: go mod vendor [-v]

Vendor resets the main module's vendor directory to include all packages
needed to build and test all the main module's packages.
It does not include test code for vendored packages.

The -v flag causes vendor to print the names of vendored
modules and packages to standard error.
$
```

__Nota bene__: "It does not include test code for vendored packages."

#### Updating Dependencies

Interestingly, `vgo` uses a fairly conservative approach to dependencies. It uses the lowest
possible version number - as opposed to the ___latest___ version number.

To list the current modules:

    $ GO111MODULE=on go list -m

To update (as in, check for more recent versions of) the current modules:

    $ GO111MODULE=on go list -m -u

There is also `go mod tidy`:

```bash
$ go mod help tidy
usage: go mod tidy [-v]

Tidy makes sure go.mod matches the source code in the module.
It adds any missing modules necessary to build the current module's
packages and dependencies, and it removes unused modules that
don't provide any relevant packages. It also adds any missing entries
to go.sum and removes any unnecessary ones.

The -v flag causes tidy to print information about removed modules
to standard error.
$
```

By default `go mod tidy` produces no output. It's worth noting that it will not clean up
any vendored dependencies either (these must be deleted manually as `go mod vendor` does
not have a removal option - the simplest option is to delete the `vendor` folder and then
re-create it with `go mod vendor`).

## GO111MODULE

It seems that the presence of a `go.mod` file implies `GO111MODULE=on` so that if
[my recommended workflow](#my-recommended-workflow) is followed, there is no need
to explicitly use `GO111MODULE=on`.

UPDATE: With Go __1.12__ this defaults to __auto__, which means that it will be
used if there is a __go.mod__ file available and will not be used if there isn't.

__NOTA BENE:__ the __auto__ setting will prevent the use of modules in the $GOPATH/src directory.

[This is fine by me but may affect ___your___ workflow.]

## My recommended workflow

This may evolve over time, but for the moment it is as follows:

1. go mod init ...
2. go mod vendor -v
3. go build ...

#### Step 1

For step 1, best results will be had by specifying the fully-qualified package name, i.e.:

    $ go mod init github.com/xxxx/yyyy

instead of:

    $ go mod init yyyy

[This will result in a cleaner dependency graph.]

#### Step 2

This will vendor the dependencies in the local file system, instead of in a central cache.

[Which is ___my___ preference.]

#### Step 3

Step 3 could just as easily be `go run ...` or `go test ...` or `make` or whatever.

## go mod edit

At present (as of Go 1.11) it seems that `go mod edit ...` does not work as expected.

A workaround is to use `sed` to edit the `go.mod` file instead.

[Remember to re-run `go mod vendor` after editing the `go.mod` file.]

## Travis CI

There are some great tips from Dave Cheney, my experience was that one of the following
will be needed to get Travis to build properly:

```
script:
  - env GO111MODULE=on go build
  - env GO111MODULE=on go test
```

[Individual environment variables]

Or else a global environment variable:

```
env:
  global:
  - GO111MODULE=on
```

[This last worked for me]

## Dependency Scanning

My initial motivation for this repo was to try out Snyk's vulnerability scanning. However,
as of this writing (September 2018) Snyk does not yet support `vgo`:

    http://support.snyk.io/hc/en-us/articles/360000911957-Language-support

[Snyk does support `dep` via `Gopkg.lock` scanning however.]

## Cloud Functions

Perhaps not surprisingly, Google's Cloud Functions support `vgo` for Go:

    http://cloud.google.com/functions/docs/writing/specifying-dependencies-go

The article recommends the following workflow:

    export GO111MODULE=on
    go mod init
    go mod tidy

[From the above page, assuming you are within the $GOPATH.]

## Reference

Probably the best place to read up on `vgo` (also known as ___go modules___):

    http://github.com/golang/go/wiki/Modules

## To Do

- [x] Install latest Golang (1.12 as of May 2019)
- [x] Investigate the use of `GO111MODULE=on`
- [x] Investigate the removal of `vgo` via `go clean`
- [x] Investigate `vgo` dependency via the use of `GO111MODULE=on`
- [x] Investigate `go mod vendor` to deal with dependencies
- [x] Investigate `go mod tidy` to deal with dependencies
- [ ] Verify dependency migration via `go mod -init`
- [x] Investigate Dave Cheney's thoughts
- [x] Add notes on Travis CI integration
- [ ] More testing
