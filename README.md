# crun

## Documentation

The user documentation is available [here](crun.1.md).

## Why another implementation?

While most of the tools used in the Linux containers ecosystem are
written in Go, I believe C is a better fit for a lower level tool like a
container runtime. runc, the most used implementation of the OCI runtime
specs written in Go, re-execs itself and use a module written in C for
setting up the environment before the container process starts.

crun aims to be also usable as a library that can be easily included in
programs without requiring an external process for managing OCI
containers.

## Performance

crun is faster than runc and has a much lower memory footprint.

This is the elapsed time on my machine for running sequentially 100
containers, the containers run `/bin/true`:

|               |    crun |   runc |       % |
| ------------- | ------: | -----: | ------: |
| 100 /bin/true | 0:01.69 | 0:3.34 | \-49.4% |

crun requires fewer resources, so it is also possible to set stricter
limits on the memory and number of PIDs allowed in the container:

```console
# podman --runtime /usr/bin/runc run --rm --pids-limit 1 fedora echo it works
Error: container_linux.go:346: starting container process caused "process_linux.go:319: getting the final child's pid from pipe caused \"EOF\"": OCI runtime error

# podman --runtime /usr/bin/crun run --rm --pids-limit 1 fedora echo it works
it works

# podman --runtime /usr/bin/runc run --rm --memory 4M fedora echo it works
Error: container_linux.go:346: starting container process caused "process_linux.go:327: getting pipe fds for pid 13859 caused \"readlink /proc/13859/fd/0: no such file or directory\"": OCI runtime command not found error

# podman --runtime /usr/bin/crun run --rm --memory 4M fedora echo it works
it works
```

crun could go much lower than that, and require \< 1M. The used 4MB is a
hard limit set directly in Podman before calling the OCI runtime.

## Dependencies

These dependencies are required for the build:

### Ubuntu

```console
$ sudo apt-get update
$ sudo apt-get install -y make git gcc build-essential pkgconf libtool \
   libsystemd-dev libcap-dev libseccomp-dev libyajl-dev \
   go-md2man libtool autoconf python3 automake
```

### Alpine

```console
# apk add gcc automake autoconf libtool gettext pkgconf git make musl-dev \
    python3 libcap-dev libseccomp-dev yajl-dev argp-standalone go-md2man
```

## Build

Unless you are also building the Python bindings, Python is needed only
by libocispec to generate the C parser at build time, it won't be used
afterwards.

Once all the dependencies are installed:

```console
$ ./configure
$ make
```

To install into default PREFIX (`/usr/local`):

```console
$ sudo make install
```

### Shared Libraries

The previous build instructions do not enable shared libraries, therefore you will be unable to use libcrun. If you wish to build the shared libraries you can change the previous `./configure.sh` statement to `./configure --enable-shared`.
