docker-rpmbuild
===============

A minimal docker rpmbuilder image.

Based on centos, includes only rpmdevtools and yum-utils and a couple
of scripts that automate building RPM packages.

The scripts take care of installing build dependencies (using
yum-builddep), building the package (using rpmbuild) and placing the
resulting RPMs in output directory.

The setup is based on Fedora packaging how-to:
http://fedoraproject.org/wiki/How_to_create_an_RPM_package

Supported tags and respective `Dockerfile` links
================================================

- [`6` (centos:6 Dockerfile)](https://github.com/jitakirin/docker-rpmbuild/blob/c6/Dockerfile) - based on [centos:6](https://registry.hub.docker.com/_/centos/)
- [`7`, `latest` (centos:7 Dockerfile)](https://github.com/jitakirin/docker-rpmbuild/blob/master/Dockerfile) - based on [centos:7](https://registry.hub.docker.com/_/centos/)

Usage
=====

The image expects that work directory will be set to directory
containing the sources (mounted from the host).

Typical usage:

```sh
docker run --rm --volume=$PWD:/src --workdir=/src \
  jitakirin/rpmbuild MYPROJ.spec
```

This will build the project `MYPROJ` in current directory, placing
results in `RPMS/${ARCH}/` and `SRPMS/` subdirectories under current
directory.

You can also specify to place the results in a subdirectory:

```sh
docker run --rm --volume=$PWD:/src --workdir=/src \
  jitakirin/rpmbuild MYPROJ.spec OUTDIR
```

This will create `OUTDIR` if necessary and place the results in
`OUTDIR/RPMS/${ARCH}/` and `OUTDIR/SRPMS/`.

If your package requires something from a non-core repo to build, you
can add that repo using a PRE_BUILDDEP hook.  It is an env variable
that should contain an inline script or command to add the repo you
need.  E.g. for EPEL do:

```sh
docker run --rm --volume=$PWD:/src --workdir=/src \
  --env=PRE_BUILDDEP="yum install -y epel-release" \
  jitakirin/rpmbuild MYPROJ.spec
```

Debugging
=========

There are two options to aid with debugging the build.  One is to set
VERBOSE option in the environment (with `-e VERBOSE=1` option to
`docker run`) which will enable verbose output from the scripts and
rpmbuild.  The other is to pass an `--sh` option to the image, which
will drop to the shell instead of running rpmbuild, e.g.:

```sh
docker run -it -e VERBOSE=1 --rm --volume=$PWD:/src --workdir=/src \
  jitakirin/rpmbuild --sh MYPROJ.spec
```

From there you can inspect the environment and you can run the build
manually either by switching to `rpmbuild` user:

```sh
su - rpmbuild
rpmbuild -ba rpmbuild/SPECS/MYPROJ.spec
```

or by running the same script the image uses:

```sh
runuser -u rpmbuild /usr/local/bin/docker-rpm-build.sh \
  ~rpmbuild/SPECS/MYPROJ.spec
```

Jenkins
=======

To use this from a Jenkins builder which itself is running under docker
(assuming it has access to host's docker socket), use something like:

```sh
docker run --rm \
  --volumes-from=JENKINS-VOLUME-CONTAINER --workdir="${WORKSPACE}" \
  jitakirin/rpmbuild MYPROJ.spec
```

This will build RPMs and place the results back in Jenkins' workspace
directory.
