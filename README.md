# vgo

![Go_Logo](/images/Go_Logo_Aqua.svg)

Getting familiar with [vgo](http://github.com/cockroachdb/cockroach/), the latest Golang package manager.

Probably the place to start is with Russ Cox's blog:

    http://research.swtch.com/vgo-tour

[UPDATE: `vgo` was released with Go 1.11 while Russ Cox's post refers to an earlier implementation.
There are substantial differences, so Russ Cox's should only be read for an overview and
not considered to be authoritative. As well, `vgo` does not seem to follow standard \*nix
conventions for command-line tools (parhaps this will change in the future).]

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
with it until there is a compelling reason to switch to `vgo`.

My personal feeling is that - while I understand the attraction of a central dependency
store - dependencies should be bundled with the code that depends on them (links can go
dead and sometimes locating dependencies can be a trial).

I will use the `main.go` file from my [UI repo](http://github.com/mramshaw/ui) for testing.

[The dependencies for this are tricky so it is a good initial test.]

## Requirements

* a recent version of Golang (preferably a released version such as 1.11 or greater)

## To Install

The command to run to install `vgo`:

    $ go get -u golang.org/x/vgo

## To Use

The command to run to build the project:

    $ vgo build

This builds a binary called `vgo` (not really what we want), so:

    $ vgo build -o greeting

We can execute this as follows:

    $ ./greeting

And everything works as expected. Huzzah!

## Git

The build generates `go.mod` and `go.sum` files. These should be checked into the Git repo.

Or perhaps not ... Dave Cheney has some interesting things to say:

    http://dave.cheney.net/2018/07/14/taking-go-modules-for-a-spin

And:

    http://dave.cheney.net/2018/07/16/using-go-modules-with-travis-ci

## Updating Dependencies

Interestingly, `vgo` uses a fairly conservative approach to dependencies. It uses the lowest
possible version number - as opposed to the ___latest___ version number.

To list the current modules:

    $ vgo list -m

To update the current modules:

    $ vgo list -m -u

## Dependency Scanning

My initial motivation for this repo was to try out Snyk's vulnerability scanning. However,
as of this writing (September 2018) Snyk does not yet support `vgo`:

    http://support.snyk.io/getting-started/languages-support

[It does support `Gopkg.lock` scanning however.]

## To Do

- [x] Install latest Golang (1.11 as of September 2018)
- [ ] Verify dependency migration via `go mod -init`
- [ ] Investigate Dave Cheney's thoughts
- [ ] More testing
