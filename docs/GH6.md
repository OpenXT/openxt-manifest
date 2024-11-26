# Legacy GHC6 Dunfell Build

This details how to setup, build, and develop against a GHC6 Dunfell
configuration of OpenXT.

## Environment Setup

> :heavy_exclamation_mark: See _Development Environment_[^1] on how to setup
> development workspace.

When using repo to setup your development workspace, below is the recommended
invocation.
```bash
   repo init -u https://github.com/OpenXT/openxt-manifest.git -m ghc6.xml
   repo sync
```

### GHC6 Haskell Compiler

Building the **dunfell** relase of OpenXT must be done under a Debian
**buster** environment. To setup a Glasgow Haskell Compiler version 6 (GHC6)
under **buster** has prerequisites that are only available from the **squeeze**
release of Debian. Below are the steps to setup GHC6 under **buster** using the
necessary libraries from **squeeze**:
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

## Building

After using the workspace setup detailed here, the remainder of the
_Development Environment_[^1] should be followed for building and developing
against the GHC6 configuration.

[^1]: [Development Environment Documentation](./Development_Environment.md)
