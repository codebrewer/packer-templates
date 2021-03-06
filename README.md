# packer-templates

Packer templates built on or inspired by [chef/bento](https://github.com/chef/bento).

## Motivation

This project was motivated by the need for a simple, bandwidth-friendly way to provision multiple Parallels VMs to build
a [Cassandra](https://cassandra.apache.org/_/index.html) cluster for general messing around. Having a base image
containing Cassandra pre-installed and needing only configuration to bring a cluster node up seemed a better option than
continually provisioning VMs from a generic base image and downloading and installing Docker and/or Cassandra each time
(or setting up local Linux package and Docker image caches).

## General Approach

The [chef/bento](https://github.com/chef/bento) repository has been added as a submodule with the goal of making it
easier to keep up to date with upstream changes than would be the case with forking and applying local changes. Check
this module out by running a command such as the following from the root of this repository:

```bash
$ git submodule update --init --recursive
```

Templates provided by Bento are copied and modified to accommodate the required changes, _e.g._:

```bash
$ mkdir -p packer_templates/ubuntu
$ cp bento/packer_templates/ubuntu/ubuntu-20.04-amd64.json packer_templates/ubuntu/ubuntu-20.04-cassandra-4.0-amd64.json
$ nano -w packer_templates/ubuntu/ubuntu-20.04-cassandra-4.0-amd64.json
```

Comparing the original and provided templates shows that some user variables have been added or modified and the paths
to the Bento-provided provisioning scripts changed so that they're found within the Git submodule when `packer` is run
in the directory containing the modified script.

## Templates Provided

### Debian 11.0 + Dockerized Cassandra

The [Debian 11.0 amd64 template](bento/packer_templates/debian/debian-11.0-amd64.json) has been used as a
[starting point](packer_templates/debian/debian-11.0-cassandra-4.0-amd64.json) and a script added to install
[Docker](https://www.docker.com) (following the process given [here](https://docs.docker.com/engine/install/debian/))
and pull [a Cassandra image](https://hub.docker.com/_/cassandra).

Other options for
[installing Cassandra](https://cassandra.apache.org/doc/latest/cassandra/getting_started/installing.html) exist but
using containers within a VM allows flexibility in setting up a cluster _e.g._ a VM could model a data centre while a
bunch of containers could model a rack.

### Ubuntu 20.04 LTS + Dockerized Cassandra

> :warning: **The Ubuntu CD image `ubuntu-20.04.1-legacy-server-amd64.iso` on which this box is based is no longer available for download**

The [Ubuntu 20.04 LTS template](bento/packer_templates/ubuntu/ubuntu-20.04-amd64.json) has been used as a
[starting point](packer_templates/ubuntu/ubuntu-20.04-cassandra-4.0-amd64.json) and a script added to install
[Docker](https://www.docker.com) (following the process given [here](https://docs.docker.com/engine/install/ubuntu/))
and pull [a Cassandra image](https://hub.docker.com/_/cassandra).

Other options for
[installing Cassandra](https://cassandra.apache.org/doc/latest/cassandra/getting_started/installing.html) exist but
using containers within a VM allows flexibility in setting up a cluster _e.g._ a VM could model a data centre while a
bunch of containers could model a rack.

## Building Boxes

The requirements for building boxes are [listed in the Bento README](bento/README.md#requirements).

To build a Debian 11.0 box including Dockerized Cassandra for only the Parallels provider:

```bash
$ cd packer_templates/debian
$ packer build -only=parallels-iso debian-11.0-cassandra-4.0-amd64.json
```

## Installing Boxes

The `install-box` script can be used to generate a suitable metadata file and add a versioned box to Vagrant. Run the
script without arguments to generate a usage message:

```bash
$ ./install-box 
Usage: install-box template provider username version

template := a relative or absolute path to a packer template
provider := the name of a Vagrant provider
username := the username under which to add the box to Vagrant
version := the version string to assign the box when adding to Vagrant
```

Example using the versioning approach inferred from Bento:

```bash
$ ./install-box packer_templates/debian/debian-11.0-cassandra-4.0-amd64.json parallels codebrewer 202109.20.0
==> box: Loading metadata for box 'builds/metadata.json'
...
==> box: Successfully added box 'codebrewer/debian-11.0-cassandra-4.0' (v202109.20.0) for 'parallels'!
```

If successful, the new box is listed along with any others already added:

```bash
$ vagrant box list
bento/ubuntu-18.04                    (parallels, 201912.04.0)
bento/ubuntu-20.04                    (parallels, 202107.08.0)
codebrewer/debian-11.0-cassandra-4.0  (parallels, 202109.20.0)
codebrewer/ubuntu-20.04-cassandra-4.0 (parallels, 202109.12.0)
```

## Licensing

This software is licensed under the Apache License Version 2.0. Please see [LICENSE](LICENSE) for details.
