# openxt-manifest
OpenXT repo tool manifest

# Quick Start
## Prepare environment
You will need to have repo tool and docker installed in your build environment. Follow the directions from [AOSP](https://source.android.com/source/downloading.html#installing-repo) to install it. Use your distribution's method for installing Docker and giving yourself permision to manage docker,
e.g. add to docker group on Ubuntu/Debian.

Using repo you will now stage your development workspace.See __Directory layout__ for details.
```
   mkdir openxt
   cd openxt
   repo init -u https://github.com/apertussolutions/openxt-manifest.git --no-clone-bundle
   repo sync --no-clone-bundle
```
Source the host-env script to configure your shell environment and finalize the staging of the development workspace.
```
  source host-env
```
Setup an OpenXT build container.
```
  create_container oxt-builder openxt-oe32
```
Now attempt a build.
```
  oxt_build [optional build id]
```
# Directory layout
Below is what the development workspace will look after it is fully initialized.
```
openxt
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

# Development Process

