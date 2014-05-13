# build-minder

``build-minder`` is a continuous integration utility for listening to webhooks and running builds.


## Development Status

Please be aware that we are very early in development. Our current focus is iterating on goals,
interfaces, architecture and some light prototyping/implementation sketching.

Until we reach a ``1.0`` version we do not intend to make any interface or configuration guarantees,
but given the overall simplicity of what we're trying to accomplish here, it probably won't be a major
issue.


## Why another continuous integration / build tool?

Continuous integration is one of the major enablers for an engineering team or process,
but it's still a bit painful to get started, and there are a handful of pitfalls that
almost every continuous integration rollout encounters (mostly revolving around not
being able to recreate the build server when it inevitably fails or has to be recreated).

``build-minder`` tries to address both of those.

It aims to simplify setup by being minimal and lightweight: it's a single binary whose primary
obligation is to route a webhook to a build script. It should be small and simple enough to
run on your laptop and on your [t1.micro](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts_micro_instances.html).

Likewise, it aspires to reduce common CI failures by removing common risk vectors.
Rather than allowing editing builds through a UI, you'll only be able to manage them
through configuration files, with an optimized simplest-and-best path of integrating
with a Git repository for storing all configuration and build scripts.


## Guiding Principles for Implementation and Design

There are probably more principles here than strictly useful, but
we can edit down over time. In order of priority they are:

1. **Usability over all else.** Our most important goal is to be easy to install, configure, maintain and use.
    When two goals come into conflict, we should always do the easiest thing for the end user.
2. **Glue, not gunk.** We should actively avoid coupling implementation to specific tools and services.
    (We will consider the tension between decoupling and usability on a feature by feature basis.)
3. **Lightweight to run and install.** You should be able to run it easily on your laptop, within a [Vagrant](http://vagrantup.com/), or on a
    [t1.micro](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts_micro_instances.html).
4. **Easily recreated build server.** Avoid all forms of transient configuration, as transient configuration is easily lost.
    All configuration (including the build scripts) should come from config files where the canonical copy is stored in remote
    version control (e.g. the build server's configuration files are a duplicate of configuration stored elsewhere,
    and if you lost every piece of configuration on the build server you could simply reclone the canonical repository).
5. **Conversative defaults.** If someone does not modify the defaults, they may not get everything they want, but it should work and should not crash.
    Example: by default we have aggresively truncate the events log rather than try to store a full history because we don't want to run out of disk space
    (which is a fairly common problem in Jenkins deployments).
6. **SOA Aware.** I'm not sure what this really means, but we'll often be running in a SOA environment, and if there are
    any particular considerations that should entail, we'd like to make those considerations.

One can only hope that we won't need even more guiding principles, this is already pretty horrifying.


## Technologies

The current set of technologies used for ``build-minder`` are:

1. [Go](golang.org) - as the general implementation language. Chosen for ability to build small, self-contained binaries with few dependencies, and reasonable performance
    for a mostly asynchronous workload (receiving webhooks, sending webhooks, calling out to shell).
2. [Git](http://git-scm.com/) - is the prefered mechanism for storing configuration and build files.
3. [LevelDB](https://code.google.com/p/leveldb/) - for storing event logs and what not. Not entirely confident in this choice, but basically want something that doesn't require end-user installation
    will compress the data it stores, and doesn't introduce much complexity.


## High-Level Architecture

![High-level architecture](docs/high-level-arch.png)

