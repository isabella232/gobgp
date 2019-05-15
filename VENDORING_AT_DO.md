# Vendoring

At this time, both gobgp and DO's internal code have been converted to use go
mod vendoring. At the time of this writing, go mod was an experimental feature
of the language and still somewhat difficult to work with.

## DO Customization

This version of gobgp is based on upstream v2.26.0 but has a bug fix for
monitoring peers. To see the change, you can do something to this effect:

    $ git clone git@github.com:osrg/gobgp.git
    $ cd gobgp
    $ git remote add do git@github.com:digitalocean/gobgp.git
    $ git fetch do
    $ git log --oneline origin/master..do/master

It is [posted as a PR to the upstream project][upstream-pr]. However, the
underlying issue may be fixed in another way as proposed by Tomo, the project
maintainer.

## Go Mod Vendoring

The final couple of custom commits remove the `go.mod` and `go.sum` files and
add this file. Removing the go mod files eases the pain of vendoring gobgp into
our internal code. The pain was mostly caused because gobgp specifies newer
revisions of many libraries which had already been vendored for other services
(since the services share a single vendoring space). It seemed that these
specified versions were arbitrarily requested by the go mod tool and were not
strictly necessary. After removing these files, the go mod file did not update
these dependencies and only pulled in one or two new dependencies needed by the
gobgp code. This was far more palatable than updating dependencies for all of
the other services unnecessarily.

Updating the vendored version can be difficult because gobgp has not been fully
converted to [modules][1]. Spcefically, it is at v2 and [doesn't use proper
versioned imports][2]. Here is the procedure to update it.

1. Edit `docode/src/do/go.mod`. Find the line near the bottom which begins with
   the following and change the tag.

    replace github.com/osrg/gobgp => github.com/digitalocean/gobgp <tag>+incompatible

2. Then, update vendoring and tidy up

    $ go mod vendor
    $ go mod tidy

[upstream-pr]: https://github.com/osrg/gobgp/pull/2128
[1]: https://golang.org/ref/mod#modules-overview
[2]: https://github.com/osrg/gobgp/issues/2274
