# NetBSD Build Guide

Updated for NetBSD [9.2](https://netbsd.org/releases/formal-9/NetBSD-9.2.html).

This guide describes how to build bitcoind, command-line utilities, and GUI on NetBSD.

## Preparation

### 1. Install Required Dependencies

Install the required dependencies the usual way you [install software on NetBSD](https://www.netbsd.org/docs/guide/en/chap-boot.html#chap-boot-pkgsrc).
The example commands below use `pkgin`.

```bash
pkgin install autoconf automake libtool pkg-config git gmake boost libevent

```

See [dependencies.md](dependencies.md) for a complete overview.

### 2. Clone Bitcoin Repo

Now that `git` and the required dependencies are installed, let's clone the Bitcoin Core repository to a directory. All build scripts and commands will run from this directory.

```bash
git clone https://github.com/bitcoin/bitcoin.git
```

### 3. Install Optional Dependencies

#### Wallet Dependencies

It is not necessary to build wallet functionality to run bitcoind or the GUI.

###### Legacy Wallet Support

`db4` is required to enable support for legacy wallets. Skip if you don't intend to use legacy wallets.

```bash
pkgin install db4
```

###### Descriptor Wallet Support

`sqlite3` is required to enable support for [descriptor wallets](https://github.com/bitcoin/bitcoin/blob/master/doc/descriptors.md). Skip if you don't intend to use descriptor wallets.

```bash
pkgin install sqlite3
```

#### GUI Dependencies

Bitcoin Core includes a GUI built with the cross-platform Qt Framework. To compile the GUI, we need to install `qt5`. Skip if you don't intend to use the GUI.

```bash
pkgin install qt5
```

The GUI can encode addresses in a QR Code. To build in QR support for the GUI, install `qrencode`. Skip if not using the GUI or don't need QR code functionality.

```bash
pkgin install qrencode
```

#### Test Suite Dependencies

There is an included test suite that is useful for testing code changes when developing.
To run the test suite (recommended), you will need to have Python 3 installed:

```bash
pkgin install python37
```

### Building Bitcoin Core

**Note**: Use `gmake` (the non-GNU `make` will exit with an error).


### 1. Configuration

There are many ways to configure Bitcoin Core. Here are a few common examples:
##### Wallet (BDB + SQlite) Support, No GUI:

This explicitly enables legacy wallet support and disables the GUI. An error will be thrown if `db4` is not installed. If `sqlite3` is installed, then descriptor wallet support will also be built.

```bash
./autogen.sh
./configure --with-gui=no \
    CPPFLAGS="-I/usr/pkg/include" \
    LDFLAGS="-L/usr/pkg/lib" \
    --with-boost-libdir=/usr/pkg/lib \
    MAKE=gmake
```

##### Wallet (only SQlite) and GUI Support:

This explicitly enables the GUI and disables legacy wallet support. If `qt5` is not installed, this will throw an error. If `sqlite3` is installed then descriptor wallet functionality will be built. If `sqlite3` is not installed, then wallet functionality will be disabled.

```bash
./autogen.sh
./configure --without-bdb --with-gui=yes \
    CPPFLAGS="-I/usr/pkg/include" \
    LDFLAGS="-L/usr/pkg/lib" \
    --with-boost-libdir=/usr/pkg/lib \
    MAKE=gmake
```

##### No Wallet or GUI

```bash
./autogen.sh
./configure --without-wallet --with-gui=no \
    CPPFLAGS="-I/usr/pkg/include" \
    LDFLAGS="-L/usr/pkg/lib" \
    --with-boost-libdir=/usr/pkg/lib \
    MAKE=gmake
```

##### Note on GCC

We currently require GCC 8.1 or greater to compile; NetBSD currently ships with an older version. You should upgrade your GCC and then pass this new version to the configure script.

For example, you could grab `gcc9` with pkgin:
```
pkgin install gcc9
```

Then, when configuring, pass the following:
```bash
./configure
    ...
    CC="/usr/pkg/gcc9/bin/gcc"
    CXX="/usr/pkg/gcc9/bin/g++"
    ...
```

### 2. Compile

Build and run the tests:

```bash
gmake # use "-j N" here for N parallel jobs
gmake check # Run tests if Python 3 is available
```
