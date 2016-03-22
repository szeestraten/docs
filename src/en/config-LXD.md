Title: Juju LXD local provider  
TODO: Remove Note re Juju only supporting 'local' (by beta3?)


# Configuring for LXD

Here we provide an overview for the creation of a controller for LXD
(see [Controllers](./controllers.html)). If your objective is instead
to create a LXD model please see [Defining a model](./models-defining.html).

Unlike other providers, with LXD, Juju does not need to be supplied with any
information regarding configuration or credentials. This makes it extremely
easy to start using LXD with Juju.


## LXD overview

The LXD local provider is the most efficient and featureful way to use Juju
locally on Ubuntu. It leverages [LXD](https://linuxcontainers.org/lxd/) to
enable multiple Juju-deployed services within a single operating system.
Immediate uses include:

 - Evaluation of Juju
 - Evaluation of deployed software
 - Experimentation with various service configurations
 - Charm development

It is a replacement for the LXC local provider supported by earlier versions
of Juju and has some notable differences:

 - The state server is no longer localhost but just another LXC container.
   The benefits of keeping installed software/files isolated in a container far
   outweigh the extra time needed to create this container.
 - Due to the above, superuser privileges (sudo) are no longer required.
 - There is no more "Juju cache" for images. LXD now has its own cache
   (`/var/lib/lxd/images`) which is separate from the traditional LXC cache
   (`/var/cache/lxc`).

Although this document does provide enough information to get you running with
LXD itself (this document is ultimately about running LXD with Juju) it does
so assuming you are setting up LXD for the first time.


## Prerequisites

 - The LXD local provider will not work on Ubuntu 12.04 LTS (Precise) and
   backporting to Ubuntu 14.04 LTS (Trusty) is incomplete.

 - Ubuntu 15.10 (Wily) or Ubuntu 16.04 LTS (Xenial) is needed.

 - The LXD stable PPA is needed on Ubuntu 15.10 (Wily).

 - Container migration (copying/moving) requires a modern version of the
   *criu* binary and a 4.4 Linux kernel (Xenial).

Proceed to install the software.

On 15.10, add this:

```bash
sudo add-apt-repository -y ppa:ubuntu-lxc/lxd-stable
sudo apt-get update
sudo apt-get install lxd
```

Now either log out and log in again to get your group membership refreshed, or
use:

```bash
newgrp lxd
```

Ubuntu 16.04(Xenial) is shipped with LXD pre-installed.


## LXD and images

LXD is image based: all LXD containers come from images and any LXD daemon
instance (also called a "remote") can serve images. When LXD is installed a
locally-running remote is provided (Unix domain socket) and the client is
configured to talk to it (named 'local'). The client is also configured to talk
to several other, non-local, ones (named 'ubuntu', 'ubuntu-daily', and
'images').

!!! Note: Juju currently only supports 'local'. Because of this, the
intermediary step of importing a desired image into 'local' is necessary.

An image is identified by its fingerprint (SHA-256 hash), and can be tagged
with multiple aliases. Juju looks for images with aliases in the format
ubuntu-&lt;codename&gt;, for instance 'ubuntu-trusty' or 'ubuntu-xenial'.

The first time an image is needed the local remote will supply it and any
subsequent requests will be satisfied by the LXD cache.

Image cache expiration and image synchronization/updates are built-in.

For any image-related command, an image is specified by its alias or by its
fingerprint. Both are shown in image lists. An image's filename is its *full
fingerprint* while an image list displays its *partial fingerprint*. Either
type of fingerprint can be used to refer to images.


### Import an image into the local remote

Import a 64bit Trusty image and create the alias:

```bash
lxd-images import ubuntu trusty amd64 --alias ubuntu-trusty
```

Such an invocation will pull official Ubuntu cloud images from
http://cloud-images.ubuntu.com.

Once any given image is imported it is cached at
`/var/lib/lxd/images` and shows up with `lxc image list`:

![lxc image list after importing](./media/image_list-imported_image-reduced70.png)


## LXD test

Although the objective here is to have Juju launch containers as it needs them,
as a test for your LXD installation, you should manually launch one now (here
called 'ubuntu-trusty-64-test').

We refer to the alias created during importation.

```bash
lxc launch ubuntu-trusty ubuntu-trusty-64-test
```

List all containers and then remove the test container:

```bash
lxc list
lxc delete ubuntu-trusty-64-test
```

!!! Note: Do not confuse command `lxc` with the binary shipped with traditional
LXC. All the latter's binaries are of the form `lxc-<subcommand>`. The `lxc`
binary actually comes from the `lxd-client` package.


## Create controller

It is time to create the controller for LXD. See
[Creating a controller](./controllers-creating.html).

This will result in the controller being visible with the LXC client:


```bash
lxc list
```

![bootstrap machine 0 in LXC CLI](./media/config-lxd_cli-machine_0.png)


## Logs

LXD itself logs to `/var/log/lxd/lxd.log` and Juju machines created via the
LXD local provider log to `/var/log/lxd/juju-{uuid}-machine-{#}`. However,
the standard way to view logs is with the `juju debug-log` command. See
[Viewing logs](./troubleshooting-logs.html) for more details.


# Useful client commands

There are many client commands available. Some common ones, including those covered
above, are given below.

<style> table td{text-align:left;}</style>

| client commands                               | meaning                            |
|----------------------------------------------|-----------------------------------|
`lxc launch`					                              | creates an LXD container
`lxc list`				                                	| lists all LXD containers
`lxc delete`					                              | deletes an LXD container
`lxc remote list`				                          | lists remotes
`lxc info`					                                | displays status of localhost
`lxc info <container>`			                     	| displays status of container
`lxc config show <container>`			               | displays config of container
`lxc image info <alias or fingerprint>`		      | displays status of image
`lxc exec <container> <executable>`	          	| runs program on container
`lxc exec <container> /bin/bash`		             | spawns shell on container
`lxc file pull <container></path/to/file> .` 	 | copies file from container
`lxc file push </path/to/file> <container>/`  	| copies file to container
`lxc stop <container>`				                     | stops container
`lxc image alias delete <alias>`		             | deletes image alias
`lxc image alias create <alias> <fingerprint>`	| creates image alias

Use `lxc --help` for more on client usage and `lxd --help` for assistance with the daemon.
See upstream documentation for how to
[configure the lxd daemon and containers](https://github.com/lxc/lxd/blob/master/doc/configuration.md)
.


## Additional notes

See [General configuration options](https://jujucharms.com/docs/stable/config-general)
for additional and advanced customization of your environment.