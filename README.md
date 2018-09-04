# vgo

![Go_Logo](/images/Go_Logo_Aqua.svg)

Getting familiar with [vgo](http://github.com/cockroachdb/cockroach/), the latest Golang package manager.

Probably the place to start is with Russ Cox's blog:

    http://research.swtch.com/vgo-tour

## Rationale

Dependency management in Golang has always been tricky. The original developers flagged
it as TBD and it's been problematic ever since. Building upon successful package managers
such as `Composer`, `npm`, and `pip` the Golang team came up with `vgo` - which uses a lot
of best practices.

Not everyone is happy about it though; for a useful recap of package management in Go
the following blog post is worth a read:

    http://codeengineered.com/blog/2018/golang-godep-to-vgo/

My personal feeling is that - while I understand the attraction of a central dependency
store - dependencies should be bundled with the code that depends on them (links can go
dead and sometimes locating dependencies can be a trial).

I will use the `main.go` file from my [ui](http://github.com/mramshaw/ui) repo fro testing.

## To Install

The command to run:

    $ go get -u golang.org/x/vgo

## To Use

The command to run:

    $ vgo build

This builds a binary called `vgo` (not really what we want), so:

    $ vgo build -o greeting

We can execute this as follows:

    $ ./greeting

And everything works as expected. Huzzah!

## Git

The build generates `go.mod` and `go.sum` files. These should be checked into the Git repo.

## Dependency Scanning

My initial motivation for this repo was to try out Snyk's velneraability scanning. However,
as of this writing (September 2018) Snyk does not yet support `vgo`:

    http://support.snyk.io/getting-started/languages-support

[It does support `Gopkg.lock` scanning however.]

## To Do

- [x] Install latest Golang (1.11 as of September 2018)
- [ ] verify dependency migration via `go mod -init`
