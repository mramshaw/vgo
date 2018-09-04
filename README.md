# vgo

![Go_Logo](/images/Go_Logo_Aqua.svg)

Getting familiar with [vgo](https://github.com/cockroachdb/cockroach/), the latest Golang package manager.

## Rationale

Dependency management in Golang has always been tricky. The original developers flagged
it as TBD and it's been problematic ever since. Building upon successful package managers
such as `Composer`, `npm`, and `pip` the Golang team came up with `vgo` - which uses a lot
of best practices.

Not everyone is happy about it though; for a useful recap of package management in Go
the following blog post is worth a read:

    https://codeengineered.com/blog/2018/golang-godep-to-vgo/

My personal feeling is that - while I understand the attraction of a central dependency
store - dependencies should be bundled with the code that depends on them (links can go
dead and sometimes locating dependencies can be a trial).

## To Install

The first command to run:

    $ go get -u golang.org/x/vgo

## To Do

- [x] Install latest Golang (1.11 as of September 2018)
