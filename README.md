# pb.py builds a Rabbit Universe

I find it convenient during RabbitMQ development to frequently build
all the Rabbit-related packages that I maintain or use. The `pb.py`
script helps me check out, update, and build those packages, tracking
dependencies between different components and rebuilding only what is
necessary. It also creates Debian and RPM packages where it can,
inventing version numbers based on the most recent
time-of-modification and source-code-repository version number for
each package. Finally, it creates a nicely-organised repository of
built artifacts, which can be linked in to an HTTP server's
DocumentRoot for simple publishing and can also be used directly or
via HTTP as a Debian package repository in `sources.list`.

## The output

The output of `pb.py` is contained in a single folder, `_repo`:

    _repo
    |-- binaries    For all produced binary build products
    |-- debian      A debian sources.list-usable package archive (Debian only)
    |-- logs        For logs from pb.py itself
    |-- manifests   All produced manifest files (see below!)
    `-- sources     For all produced source-code archives

## Build products are versioned

Each build product (source tarball, binary tarball, `.ez` file, deb,
rpm) is versioned. The version number is computed from (a hash of) the
precise git/mercurial revision identifiers of all of the subprojects
that contributed to the built artifact. For example,

    rabbitmq-server-generic-unix-20100822.195610.trunk.f63dc6ae.tar.gz

The version is `20100822.195610.trunk.f63dc6ae`:

 - The first and second parts are the date and time of the newest
   revision of any part of the codebase used to produce the file.

 - The third part is the name of the configuration selected when
   `pb.py` was invoked; usually, this will be "trunk" unless you have
   special requirements.

 - The fourth part is the first few digits of the hash of the
   *manifest* file for the build product. See below.

The fourth part is crucial. Because the version number contains a hash
of the manifest file for the build product, and the manifest
identifies exactly the source code that was used to build it, then
from the version number and the manifest files produced by `pb.py` we
can reconstruct the exact source code in order to, for example, track
down a bug. The manifest file for the `rabbitmq-server` version given
above would be
`_repo/manifests/rabbitmq-server-20100822.195610.trunk.f63dc6ae.manifest`.

## Manifest files

Manifest files describe exactly what went in to a particular
package. Here's an example manifest, for `presence-exchange`, a
RabbitMQ server plugin:

    Project: presence-exchange
    Description: x-presence exchange type
    Repository: git://github.com/tonyg/presence-exchange
    Changed: 1277081329 (20100621004849)
    Revision: a29d4ddff769afe0a40b364b6b543870e782afa5

    Project: rabbitmq-server
    Description: rabbitmq-server
    Repository: http://hg.rabbitmq.com/rabbitmq-server
    Changed: 1282506970 (20100822195610)
    Revision: 9fd9df35b90c

    Project: rabbitmq-erlang-client
    Description: rabbitmq-erlang-client
    Repository: http://hg.rabbitmq.com/rabbitmq-erlang-client
    Changed: 1281961949 (20100816123229)
    Revision: 8ff5f9e9b7c1

    Project: rabbitmq-codegen
    Description: rabbitmq-codegen
    Repository: http://hg.rabbitmq.com/rabbitmq-codegen
    Changed: 1281699796 (20100813114316)
    Revision: f8b34141e6cb

Here we see that `presence-exchange` is built from four
subprojects. Each subproject has

 - a name: this is the name of the directory holding the subproject's source code;
 - a description: this is a simple description of the purpose of the subproject;
 - an upstream repository URL: both git and mercurial use a URL-like syntax for repository locations;
 - a timestamp: both in seconds-since-the-epoch and more human-readable GMT format; and
 - a revision identifier: both git and mercurial supply long, unique hexadecimal revision IDs.

From the manifest file we can reconstruct precisely the set of source
code that was used to produce a given build product. `pb.py` produces
manifest files alongside build products as it goes.

## A debian package archive is produced on Debian build hosts

To use the debian archive, expose it via e.g. http and add a line like
the following to your `sources.list`:

    deb http://YOURHOST/PATH/TO/YOUR/_repo/debian/ kitten main
