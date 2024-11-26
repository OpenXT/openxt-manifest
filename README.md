# openxt-manifest

OpenXT repo tool manifest

## Quick Start

You will need to have repo tool installed in your build environment.  Follow the
directions from
[AOSP](https://source.android.com/source/downloading.html#installing-repo) to
install it.

> :heavy_exclamation_mark: See _Development Environment_[^1] on how to setup
> development workspace.

### Setup Repo Environment

Using repo you will now stage your development workspace. See __Directory
layout__ for details.
```bash
   mkdir openxt-workspace
   cd openxt-workspace
   repo init -u https://github.com/OpenXT/openxt-manifest.git
   repo sync
```

### Install Haskell Compiler

Install GHC8 for building OpenXT Haskell componnents.

```bash
wget https://downloads.haskell.org/ghc/8.10.7/ghc-8.10.7-x86_64-deb10-linux.tar.xz

tar -xf ghc-8.10.7-x86_64-deb10-linux.tar.xz
rm ghc-8.10.7-x86_64-deb10-linux.tar.xz

pushd ghc-8.10.7
./configure --prefix=/usr/local
make install
popd && rm -rf ghc-8.10.7
```

### Setup build workspace

Set up a build instance named _quickstart_ that will be located in the
directory _build-quickstart_. After following these steps your shell will be in
the build directory with all the necessary environment variables configured to
start building and developing OpenXT component and VM images.

```bash
openxt/bordel/bordel -i quickstart config --no-repo-branch
source build-quickstart/build-env
```

[^1]: [Development Environment Documentation](./Development_Environment.md)
