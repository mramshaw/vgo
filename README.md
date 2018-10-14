# vgo

![Go_Logo](/images/Go_Logo_Aqua.svg)

Getting familiar with [vgo](http://github.com/cockroachdb/cockroach/), the latest Golang package manager.

Probably the place to start is with Russ Cox's blog:

    http://research.swtch.com/vgo-tour

[UPDATE: `vgo` was released with Go 1.11 while Russ Cox's post refers to an earlier implementation.
There are substantial differences, so Russ Cox's blog should only be read for an overview
and not considered to be authoritative. As well, `vgo` does not seem to follow standard \*nix
conventions for command-line tools (perhaps this will change in the future).]

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

[Probably this is a use case for `go mod vendor` although that is just a guess for now.]

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

## Updating Dependencies

Interestingly, `vgo` uses a fairly conservative approach to dependencies. It uses the lowest
possible version number - as opposed to the ___latest___ version number.

To list the current modules:

    $ go list -m

To update the current modules:

    $ go list -m -u

## Dependency Scanning

My initial motivation for this repo was to try out Snyk's vulnerability scanning. However,
as of this writing (September 2018) Snyk does not yet support `vgo`:

    http://support.snyk.io/getting-started/languages-support

[It does support `dep` via `Gopkg.lock` scanning however.]

## To Do

- [x] Install latest Golang (1.11 as of September 2018)
- [x] Investigate the use of `GO111MODULE=on`
- [x] Investigate the removal of `vgo` via `go clean`
- [x] Investigate `vgo` dependency via the use of `GO111MODULE=on`
- [ ] Investigate `go mod vendor` to deal with dependencies
- [ ] Investigate `go mod tidy` to deal with dependencies
- [ ] Verify dependency migration via `go mod -init`
- [ ] Investigate Dave Cheney's thoughts
- [ ] More testing
