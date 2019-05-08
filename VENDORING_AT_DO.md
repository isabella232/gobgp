## Go Mod Vendoring

1. Rebase the latest [DO tag][tags] onto a new upstream tag.

    $ git switch -c update-do <latest-do-tag> (e.g. v2.29.0-do.1)
    $ git rebase <latest-upstream-tag> (e.g. v2.30.0)

2. Make any local modifications necessary, clean up (using rebase -i), and tag
   the new release (e.g. v2.30.0-do.1)

    $ git tag <tag>
    $ git push do <tag>

1. Edit `docode/src/do/go.mod`. Find the line near the bottom which begins with
   the following and change the tag.

    replace github.com/osrg/gobgp => github.com/digitalocean/gobgp <tag>+incompatible

2. Then, update vendoring and tidy up

    $ go mod vendor
    $ go mod tidy

## DO Customizations

You can see all of the commits that we carry locally with the following command.

    $ git log origin/master..@

    - or, if you don't have this checked out in your current working dir -

    $ git log origin/master..<latest-do-tag> (e.g. v2.30.0-do.1)

1. One patch from Neal which is required for EVPN type 5 routes to work.

[tags]: https://github.com/digitalocean/gobgp/tags
