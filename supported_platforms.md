We support a pretty wide set of platforms (Windows, Linux (glibc and musl), macOS, FreeBSD) and architectures (x86_64, i686, armv7l, aarch64, ppc64le) where applicable. Yes, we did in fact look into GHA at a while back, but for us the dealbreaker was incomplete runner platform support. With a single CI system, we centralize developer effort in developing plugins (such as those we wrote for secrets management in public repositories, or for launching dynamic jobs based on the diff of a PR).

The full list of platforms-architectures we build for are:

Linux:

- glibc
   - x86_64
   - i686
   - aarch64
   - arm7I
   - powerpc64le
   - armv6l is in the works
- musl
   - x86_64 (other arches pending demand)

Windows:
- x86_64
- i686

macOS:
- x86_64
- aarch64

FreeBSD
- x86_64 (other pending demand)

We’ve built up a set of technologies that allow us to do things like nest containers on Linux, allowing us to run the CI agent in one container, which then spawns child containers with the rootfs images for specific build steps, allowing us to have very generic workers that download all the necessary toolchains, etc. on demand, and cached appropriately. The Julia language needs a rather large set of compilers and support libraries to build, and our system allows us to run our own workers and replicate the secure, almost-zero-configuration environment.

Our workloads are quite demanding, downloading gigabytes of data per-job that can be 99% cached. As a result, our system has a persistent, on-worker cache.

The specifications of our workers vary greatly per platform. In general, we try to provision a minimum of 2GB of RAM per core, with up to 8 cores per worker. Our armv7l boards are, of course, much smaller at 4GB of RAM total, and our x86_64 workhorse machines are giant AMD EPYC machines with 500GB of RAM and 128 cores (we run multiple workers on a single node for these large machines, and in some ways, this is added complexity that we don't want; we'd much rather run things on dedicated, 8-core 16GB machines). Disk performance is somewhat important to us, as we generally spend 1-2 minutes of CI time compiling C code, and we accelerate that with `ccache`. Having multiple cores is primarily useful for our test suites, as those are well parallelized and are CPU bound. So we generally run a packager and a tester on the same machine, with the former building Julia (which is, for most builds, disk-bound due to `ccache`, followed by a long-running serial process performing julia bootstrap) and the latter running the parallelized tests, which can take quite a while, even split across 8 cores.
