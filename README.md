# openxt-manifest

OpenXT repo tool manifest

## Quick Start

You will need to have repo tool and docker installed in your build environment.
Follow the directions from
[AOSP](https://source.android.com/source/downloading.html#installing-repo) to
install it. Use your distribution's method for installing Docker and giving
yourself permision to manage docker, e.g. add to docker group on Ubuntu/Debian.

Using repo you will now stage your development workspace.See __Directory
layout__ for details.
```bash
   mkdir openxt-workspace
   cd openxt-workspace
   repo init -u https://github.com/apertussolutions/openxt-manifest.git
   repo sync
```

Source the host-env script to configure your shell environment and finalize the
staging of the development workspace.
```bash
  source host-env
```

Setup an OpenXT build container.
```bash
  # create_container <container-name> <base-container-image>
  create_container oxt-builder openxt-oe32
```

Generate certs for signing the build
```bash
  # gen-certs <output-directory>
  ./bin/gen-certs certs
```

Now attempt a build.
```bash
  # oxt_build <container-name> [optional-build-id]
  oxt_build oxt-builder
```

To restart or rebuild a previous build (e.g. build-123456), supply the build id.
```bash
  # oxt_build <container-name> [optional-build-id]
  oxt_build oxt-builder 123456
```

When the build is complete, generate a release, sign it, and crete an ISO
```bash
  # gen-iso -n <build-id> -r <release-tag>
  ./bin/gen-iso -n 123456 -r 1-stable
```
The ISO will end up at `openxt-workspace/build-{build-id}/release/{release-tag}/installer.iso`

## Development Environment

There are two environments, host and build. The host environment is setup
through the sourcing of the ```host-env``` file into your bash environment. The
build environment is within a docker container and is setup through the
sourcing of the ```openxt-env``` file.  These environment files ensure the
directory structure is setup in an expected manner and loads environment
variables and functions into your shell environment.

### Host Environment

The host environment is considered to be the directory in which the repo tool
was executed.

#### Workspace

After sourcing the ```host-env``` script the host development workspace will
have a directory structure as seen below.
```
openxt-workspace
├── bin/
├── Dockerfiles/
├── host-env
├── mirror -> .repo/projects/openxt/
├── openxt/
├── openxt-env
├── .repo/
├── repos/
└── templates/
```

The scope of each directory is as follows,
* **bin**: contains helper shell scripts
* **Dockerfiles**: contains Docker container definitions
* **mirror**: this is a symlink to the bare git repositories setup by repo tool
* **openxt**: contains git working trees for each of the bare repositories in mirror
* **repos**: contains the OE layers available for use in a build
* **templates**: contains configuration fragments used to create OE configuration

##### Git Setup

This manner in which the git repositories are setup by the repo tool and the
*host-env* source file is very specific to enable rapid code/build/test
cycles.

###### Working Tree

Repo tool uses git's working trees capability to maintain a local bare mirror
of each source repository and then sets up a working tree for each repository
in the *openxt* directory. The *host-env* file takes advantage of this
situation and creates a symlink named *mirror* that points to the directory
containing the bare repositories.  The result is that any code changes commited
in a working tree under *openxt* is immediately reflected in the bare
repository available under *mirror*.

The build environment then uses this situation to populate the
**OPENXT_GIT_MIRROR** variable with the *mirror* directory. The result is a
self contained environment where a developer can make code changes, commit to
the working branch, and then immediately enter the build container (if they are
not already in the container) to build the recipe for the respective code base.

###### Working Branch

Repo tool provides the ability to create a branch across all code repositories
under its control. The *host-env* file uses this to create a branch, *build* by
default, that is used by the build environment for the **OPENXT_BRANCH**
variable. This enables branch naming consistency allowing for the development
of features that crosses OpenXT source repositories without acts of
contortionims to execute a build.

#### Workspace Shell Functions

To ease working in the environment a set of shell functions were provided
through the ```host-env``` file.

##### **create_container** *{name}* *{file}*  

Create a container with the name *{name}* using the Dockerfile *{file}*.
The Dockerfile *{file}* must be present in the Dockerfiles directory. The
provided Dockerfiles have been crafted to enable mapping your local UID and GID
to those of the build user within the container. This shell function will pass
the UID and GID of the user that invoked the function to Docker as the UID and
GID of the build user.

##### **create_oebuild** *{build_id}* *{template}*

Create a build directory with the build id of *{build_id}* and configuration
generated using *{template}* bundle of configuration fragments. In building
each OE configuration file, the function will look for a file name *{OE
configuration file}.{template}*.

##### **enter_container** *[-u USER]* *{name}*

This is a wrapper function to start the build container *{name}* that has been
configured as expected by the build environment as well as providing an
environment that can support development activities. By default this shell
function assumes that the standard user build was setup when the container was
created, but the *-u* flag is provided to optionally override the user name.
The successful execution of this function will result in an interactive shell
environment within the container as the build user in the build user's home
directory.

##### **oxt_build** *[-u USER]* *{name}* *[build-id]*

This shell function is simliar to **enter_container** except instead of
launching an interactive session in the container, it launches the build script
located in the bin directory.

#### Workspace Helper Scripts

There are a set of helper/utility scripts in the *bin* directory available to
help with different aspects of the development and build cycles.

##### **build-oxt**

This script is written to execute a complete build of OpenXT. It is the script
used by the **oxt_build** shell function.

##### **gen-certs** *[-s "x509 Distinguished Name"]* *[path]*

This will generate a set of production and development x509 certificate and key
pairs using the traditional OpenXT Distinguished Name (DN). The DN may be
overridden using the *-s* flag. The certs and keys will be stored in
${HOME}/certs unless an alterntive is provided with *path* parameter.

**NOTE**: *The build environment will expect these to be in the directory
certs in the host environment.*

##### **gen-release** *{build_id}* *{target_dir}*

After a build with id *{build_id}* completes, this script can be ran passing a
*{target_dir}* and an OpenXT release ISO tree will be constructed within the
directory.

##### **sign-oxt-repo** *{build_id}* *{target_dir}*

This script is used in conjunction with **gen-release**. Pass the same
parameters and the release package will be signed with the build certs.


##### **gen-iso** -n *{build_id}* -r *{release_tag}*

This script is a wrapper around gen-release, sign-oxt-repo, and a couple
of other commands to prepare the final .iso image file. It takes two options
   -n : the build id
   -r : the  release tag

The genearted iso file is placed under build-<BUILD_ID>/release/<RELEASE_TAG>

### Build Environment

The build environmnet is a pseudo-ephemeral environment that is Docker
container with persistent directory structure mapped into the container.

#### Workspace

Upon entering the build container using the *enter_container* function, the
persistent directory structure can be seen below.

```
${HOME}
├── .ssh/
├── .gitconfig
├── .repoconfig
├── .repo_.gitconfig.json
└── openxt/
```

The dot files are the user's dot files from the host machine while the *openxt*
directory is the host environment.

##### Build Setup

To run a build within the environment a build directory must be setup. This can
be done in two ways,

1. Use the **create_oebuild** from the host environment to create the build
   directory
2. Let the openxt-env source file create one

**NOTE**: *The build environment expects the build directory to be under the
openxt directory.*

To enter the build setup,source the *openxt-env* file from within the *openxt*
directory. The *openxt-env* file takes a parameter *build_id* that can be used
for one of the following,

1. To override the build id used for the build
2. To enter an existing build setup, either pre-created or previous build

When *openxt-env* completes the shell will be in the build directory with
environment variables setup to run an OE build.

##### Building

Due to OpenXT's 32bit dependency, bitbake cannot be invoked directly. To
handle this, the *openxt-env* source file exports the bb shell function. This
function uses the linux32 command to deceives bitbake in to believe it is
running under a 32bit kernel.

Within this setup, it is possible to build any OpenXT recipe for any OpenXT
machine type. The general invocation would look like,

```
 MACHINE={OpenXT Machine} bb {recipe name}
```
