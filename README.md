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
