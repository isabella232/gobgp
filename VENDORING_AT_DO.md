# Vendoring

At this time, both gobgp and DO's internal code have been converted to use go
mod vendoring. At the time of this writing, go mod was an experimental feature
of the language and still somewhat difficult to work with.

## DO Customization

This version of gobgp is based on upstream v2.8.0 but has a bug fix for
monitoring peers. To see the change, you can do something to this effect:

    $ git clone git@github.com:osrg/gobgp.git
    $ cd gobgp
    $ git remote add do git@github.com:digitalocean/gobgp.git
    $ git fetch do
    $ git log --oneline origin/master..do/master

It is [posted as a PR to the upstream project][upstream-pr]. However, the
underlying issue may be fixed in another way as proposed by Tomo, the project
maintainer.

To update to a newer upstream version of gobgp, you need to rebase the local
modifications to the new upstream version. For example, to update to v2.19.0, do
the following (assuming you've cloned the repository as shown above).

    $ version=v2.19.0
    $ git fetch
    $ git fetch do
    $ git checkout -B master do/master
    $ git rebase ${version}

You're likely to hit some conflicts in this process. If the conflict is in the
protobuf generated files (e.g. `api/gobgp.pb.go`) then you can clobber them and
regenerate. You will need to download the exact version of protobuf that gobgp
wants which was version 3.7.1 at the time of this writing.

    $ tools/grpc/genproto.sh
    $ git add api/gobgp.pb.go
    $ git rebase --continue

If you hit other merge conflicts, you'll have to work it out. Once you have
successfully rebased the local modifications onto the new 

    $ git tag "${version}-do.1"
    $ git push do "${version}-do.1"

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

[upstream-pr]: https://github.com/osrg/gobgp/pull/2128

To update the version of gobgp, first rebase local modifications onto the new
upstream version as described in the section above. Then, update the vendored
version like this.

    $ GO111MODULE=on go mod edit \
        -replace=github.com/osrg/gobgp=github.com/digitalocean/gobgp@${version}-do.1
    $ GO111MODULE=on go mod vendor
    $ GO111MODULE=on go mod tidy
    $ git add vendor
    $ git commit

Test the hvannounced binary using the new version.
