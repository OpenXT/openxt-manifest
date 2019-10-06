# openxt-manifest

OpenXT repo tool manifest

## Quick Start

You will need to have repo tool installed in your build environment.  Follow the
directions from
[AOSP](https://source.android.com/source/downloading.html#installing-repo) to
install it.

Using repo you will now stage your development workspace. See __Directory
layout__ for details.
```bash
   mkdir openxt-workspace
   cd openxt-workspace
   repo init -u https://github.com/apertussolutions/openxt-manifest.git
   repo sync
```

## Development Environment

NOTE: The default non-interactive shell for OpenXT build is supposed to be `dash`
      However, some scripts make use of `bash` extentions. While this should be fixed,
      a work around for now is to change the default shell via:
```bash
      dpkg-reconfigure dash
```
### Directory Structure

After running repo sync, the development environment will have a directory
structure as seen below.
```
openxt-workspace
├── layers
├── mirror -> .repo/projects/openxt
└── openxt
```

The scope of each directory is as follows,
* **layers**: contains the OE layers available for use in a build
* **mirror**: this is a symlink to the bare git repositories setup by repo tool
* **openxt**: contains git working trees for each of the bare repositories in mirror

### Git Setup

This manifest leverages repo tool's approach to environment setup to
enable rapid code/build/test cycles for OpenXT.  In particular it
provides a self contained environment within a single directory where a
developer can easily have multiple code changes in progress while having
one or more builds running without any conflicts with themselves or
others on a shared system.

#### Working Tree

Repo tool uses git's [working trees](https://git-scm.com/docs/git-worktree)
capability to maintain a local bare mirror of each source repository and then
sets up a working tree for each repository in the `openxt` directory. The
manifest takes advantage of this situation and creates a symlink named
`mirror` that points to the directory containing the bare repositories.  The
result is that any code changes commited to the working tree under `openxt` is
immediately reflected in the bare repository available under `mirror`.

The OpenXT build recipes uses the **OPENXT_GIT_MIRROR** variable to
locate Git repository server containing the OpenXT code repositories.
This variable can be set to point at `mirror` directory using the
`file://` URI. This results in builds pulling code that is committed
locally without the need of an intermediate Git repository server.

#### Working Branch

Repo tool provides the ability to create a branch across all or a subset
of code repositories under its control. The OpenXT build recipes use the
**OPENXT_BRANCH** variable to select what branch to pull from the
repositories held by **OPENXT_GIT_MIRROR** repository server. Therefore
in an **openxt-manifest** setup a developer can quickly create one or
more working branch across all the OpenXT code repositories, switch to
working branch then need to develop on, commit changes to the branch,
launch a build on that branch, and then optionally switch to different
working branch to make changes without affecting any currently executing
builds.


```
repo start ${OPENXT_BRANCH} openxt/*
```

## Building

### Build Machine Setup

#### OE Setup
OpenXT uses the OpenEmbedded build system to build itself. The first
step in setting up a build machine is to ensure the machine is
configured for OpenEmbedded. Follow the appropriate directions on
OpenEmbedded's [Getting
started](https://www.openembedded.org/wiki/Getting_started) wiki
page.

#### Debian Based Environments

Currently the primary distributions used for building are Debian based.

##### OpenXT Additional Packages

These are additional packages recommend be installed
```
bcc
bin86
curl
cvs
docbook-utils
dosfstools
genisoimage
guilt
iasl
libelf-dev
libncurses5-dev
libsdl1.2-dev
liburi-perl
locales
mtools
openssl
policycoreutils
python-pysqlite2
quilt
rpm
subversion
xorriso
```

##### Haskell Compiler

Building OpenXT requires a specific version of the Glasgow Haskell
Compiler (GHC) to compile the OpenXT's Haskell code. The prerequisites
for this version are only availbe from the **squeeze** release. Below
are the steps to setup GHC,
```
# Install the prerequisites for GHC
mkdir -p ghc-prereq && pushd ghc-prereq

wget http://archive.debian.org/debian/pool/main/g/gmp/libgmpxx4ldbl_4.3.2+dfsg-1_amd64.deb
wget http://archive.debian.org/debian/pool/main/g/gmp/libgmp3c2_4.3.2+dfsg-1_amd64.deb
wget http://archive.debian.org/debian/pool/main/g/gmp/libgmp3-dev_4.3.2+dfsg-1_amd64.deb

dpkg -i libgmpxx4ldbl_4.3.2+dfsg-1_amd64.deb libgmp3c2_4.3.2+dfsg-1_amd64.deb \
	libgmp3-dev_4.3.2+dfsg-1_amd64.deb

popd && rm -rf /tmp/ghc-prereq

# Install the required version of GHC
wget https://downloads.haskell.org/~ghc/6.12.3/ghc-6.12.3-x86_64-unknown-linux-n.tar.bz2

tar jxf ghc-6.12.3-x86_64-unknown-linux-n.tar.bz2
rm ghc-6.12.3-x86_64-unknown-linux-n.tar.bz2

pushd ghc-6.12.3
./configure --prefix=/usr/local
make install
popd && rm -rf ghc-6.12.3
```

### Build Certificates

OpenXT signs its build with a private key for which the public
certificate of that key is built into the core domain (Dom0). An
enterprise CA may be used to generate the certs. If one is not available
or the build is for development/personal purposes, a [self-signed
certificate](https://www.shellhacks.com/create-self-signed-certificate-openssl/)
may be used.

OpenXT build system supports two certificates, a production certificate
and a development certificate. They are identfied in the build via the
Bitbake variables,
 
* **REPO_PROD_CACERT**: path to CA certificate that signed the production signing certificate
* **REPO_DEV_CACERT**:  path to CA certificate that signed the development signing certificate

**NB**: It is convention that that the **REPO_PROD_CACERT** be called
`prod-cacert.pem` and the **REPO_DEV_CACERT** be called
`dev-cacert.pem`. It is also recommend that all the certificates are
stored together in a single directory either directly or symlinked into
the build directory.

### OpenEmbedded oe-init-build-env

It is possible to build a complete OpenXT installation using upstream
OpenEmbedded `oe-init-build-env` script.

#### Initialize a Build Directory

```
BITBAKEDIR="${PWD}/layers/bitbake" \
	source layers/openembedded-core/oe-init-build-env

# Recommended symlinking mirror to ease Bitbake variables
ln -s ../mirror
```

##### Adjusting the bblayers file

The `conf/bblayers.conf` file will need to be updated to include the
following layers
```
  meta-openembedded/meta-oe
  meta-openembedded/meta-python
  meta-openembedded/meta-networking
  meta-openembedded/meta-gnome
  meta-openembedded/meta-xfce
  meta-intel
  meta-java
  meta-selinux
  xenclient-oe
  meta-openxt-ocaml-platform
  meta-openxt-haskell-platform
  meta-virtualization
```

##### OE Local configuration

The `oe-init-build-env` script will have copied in a template
`conf/local.conf` into the new build directory. The `conf/local.conf`
file will need to be adjusted with the following additional sections.

###### Build variables

* **DISTRO**: an OE variable and should be set to `openxt-main`
* **OPENXT_MIRROR**: mirror URL for build related files hosted by OpenXT
* **OPENXT_GIT_MIRROR**: the path element of the URL for the Git repository server hosting OpenXT code repositories
* **OPENXT_GIT_PROTOCOL**: the URI element of the URL for the Git repository server hosting OpenXT code repositories
* **OPENXT_BRANCH**: the branch to checkout from all OpenXT code repositories. This should be the branch where you might have custom development code that you want to test.
* **REPO_PROD_CACERT**: see **Build Certificates** section above
* **REPO_DEV_CACERT**: see **Build Certificates** section above

**Example configuration**:
```
DISTRO ?= "openxt-main"

OPENXT_MIRROR = "http://mirror.openxt.org"
OPENXT_GIT_MIRROR = "${TOPDIR}/mirror"
OPENXT_GIT_PROTOCOL = "file"
OPENXT_BRANCH = "build-190805"

OPENXT_CERTS_DIR = "${TOPDIR}/certs"

REPO_PROD_CACERT="${OPENXT_CERTS_DIR}/prod-cacert.pem"
REPO_DEV_CACERT="${OPENXT_CERTS_DIR}/dev-cacert.pem"
```

##### Build identity

* **OPENXT_BUILD_ID**: OpenXT build ID, incremental numeral starting from the given number.
* **OPENXT_VERSION**: OpenXT version number. This will influence the upgrade path.
* **OPENXT_RELEASE**: Name Refering to the release (purely cosmetic).
* **OPENXT_UPGRADEABLE_RELEASES**: List of OpenXT versions that can be upgraded to this version.
* **OPENXT_BUILD_BRANCH**: OpenXT branch name.

**Example config**:
```
OPENXT_BUILD_ID="0"
OPENXT_VERSION="10.0.0"
OPENXT_RELEASE="agrypnotic"
OPENXT_UPGRADEABLE_RELEASES="8.0.0 8.0.1 8.0.2 9.0.0"
OPENXT_BUILD_BRANCH="master"
OPENXT_BUILD_DATE ??= "${@time.strftime('%H:%M:%S %m/%d/%y',time.gmtime())}"

# XenClient legacy variables
XENCLIENT_BUILD = "${OPENXT_BUILD_ID}"
XENCLIENT_BUILD_BRANCH = "${OPENXT_BUILD_BRANCH}"
XENCLIENT_RELEASE = "${OPENXT_RELEASE}"
XENCLIENT_VERSION = "${OPENXT_VERSION}"
XENCLIENT_BUILD_DATE = "${OPENXT_BUILD_DATE}"
```
#### Build mirrors

To minimize flux in recipes due to source code moving a set of
**MIRRORS** mappings are used remap URL in a recipe to a destination
that exists.

```
# Common URL translations.
MIRRORS += " \
    http://code.coreboot.org/p/seabios/downloads/.* \
https://www.seabios.org/downloads/ \
    http://www.seabios.org/downloads/.* \
https://www.seabios.org/downloads/ \
    git://anonscm.debian.org/collab-maint/ltrace.git  \
git://github.com/sparkleholic/ltrace.git \n \
"
```

#### Build Configuration

It is possible to control the OpenXT build to produce debug builds for
example. Below are configuration options providing those controls.

```
# Set SELinux enforcement, for debug set to "permissive"
DEFAULT_ENFORCING = "enforcing"

# Enable debugging tweaks.
EXTRA_IMAGE_FEATURES += "debug-tweaks"
```

#### Bordel

An optional build tool called **Bordel** is included in the manifest
under `openxt/bordel`. It provides an opinionated approach to setup and
manage build directories along with building and packaging OpenXT's
images in installerable formats. How to setup and use **Bordel** can be
found in its README file.

### Building the Images

To build an instance of OpenXT, there are a set of images that must be
built. The images and their respective OpenEmbedded **MACHINE** types
are below.
```
   MACHINE		    Image Name
xenclient-dom0		xenclient-initramfs-image
openxt-installer	xenclient-installer-image
openxt-installer	xenclient-installer-part2-image
xenclient-stubdomain	xenclient-stubdomain-initramfs-image
xenclient-dom0		xenclient-dom0-image
xenclient-uivm		xenclient-uivm-image
xenclient-ndvm		xenclient-ndvm-image
xenclient-syncvm	xenclient-syncvm-image
```
**NB**: The image `xenclient-dom0-image` cannot be built until the
`xenclient-initramfs-image` and `xenclient-stubdomain-initramfs-image`
images have been built.

The command to build an image is as follows,
```
 MACHINE={OpenXT Machine} bitbake {Image Name}
```
