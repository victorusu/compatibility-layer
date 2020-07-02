# Compatibility layer

## Introduction

The compatibility layer of the EESSI project uses [Gentoo Prefix](https://wiki.gentoo.org/wiki/Project:Prefix)
to provide a known base on top of the host. This is the foundation we use to build our software stack on.
An alternative would be the [NixOS](https://nixos.org/).

## Installation and Configuration

### Prerequisites

The bootstrap process will need a clean environment with a compiler (the system version of gcc will do). It also is very sensitive to 
the environment, so setup a user with unset CFFLAGS, CFLAGS, LDFLAGS, PKG_CONFIG_PATH and the always harmful LD_LIBRARY_PATH variables.

EESSI provides a Singularity container for this.

### Building the Singularity container
The provided Singularity definition file can be used to build a container with a clean environment:
```
sudo singularity build bootstrap-prefix.sif singularity-bootstrap-prefix.def
```

### Bootstrapping Gentoo Prefix
Gentoo Prefix provides a bootstrap script to build the prefix, see [Gentoo Prefix Bootstrap](https://wiki.gentoo.org/wiki/Project:Prefix/Bootstrap).
We forked [this version](https://gitweb.gentoo.org/repo/proj/prefix.git/tree/scripts/bootstrap-prefix.sh?id=050a9859dd4629a46385a24a85ecc2d195bcaf86)
of the bootstrap script and made the following modifications:

- removed the minus on line 1528 to solve a circular dependency issue;
- use https instead of http for the download mirrors on lines 2094 and 2095;
- fix line 589 and take into account that $SHELL may contain additional flags (e.g. `SHELL="/bin/bash --no-profile --norc"`).

You can run our version of the bootstrap script (see `bootstrap-prefix.sh`) inside the Singularity container by executing:
```
singularity run bootstrap-prefix.sif
```
or simply:
```
./bootstrap-prefix.sif
```

If you want to run your own version of the bootstrap script, use:
```
singularity exec bootstrap-prefix.sif ./bootstrap-prefix.sh
```

After starting the bootstrap have a long coffee...

### Adding EESSI overlay
Additional packages are added in the EESSI overlay, which is based on ComputeCanada.
To add the overlay: 

Start the prefix
```
startprefix
```
Configure the overlay
```
mkdir $(EPREFIX)/etc/portage/repos.conf
emerge eselect-repository
eselect repository add eessi git https://github.com/EESSI/gentoo-overlay.git
```
Sync the overlay
```
emerge --sync
```
