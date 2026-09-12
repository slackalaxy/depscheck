# prt-supplements

Some extra, unofficial, tools to check for dependencies and refresh ports.

## finddepsrow

List port's dependencies as a row.

Output dependencies found by `finddeps` as a single row, suitable for the "\# Depends on:" line in the Pkgfile. Dependencies from core are omitted. The wrapper also checks if any libs are missing by calling `revdep`.

## lsopt

This script simply list the optional dependencies for a port.

## missdeps

Check for missing deps of all packages that are installed.

This tool uses `prt-get listinst` to check for missing dependencies of the packages that are installed and will output a simple list of missing dependencies, easy to parse. Supplying the `-v|--verbose` option will report the missing dependencies and their immediate installed dependents:

```         
missdeps -v

-- ([ ] missing deps, [d] = dependents)
[ ] eudev
[ ] sqlite3
[d] apr-util
[d] dhcpcd
[d] libinput
[d] libusb
[d] nss
[d] openmpi
[d] pciutils
[d] python3
[d] util-linux
```

## pkg-sync
This is meant for systems managed by [pkg-get](https://crux.nu/portdb/?a=search&q=pkg-get) to install prebuilt packages. The script aims to automate the following steps:
1. Update ports, by simply calling `ports -u`.
2. Sync pkg-get repo with upstream. This will clean the repo local folder first. Its location is parsed from `/etc/pkg-get.conf`.
3. Remove orphaned ports, reported by `prtorphan`.
4. Install missing deps, reported by `missdeps`.
5. Show what will be updated by `pkg-get diff`.
6. Update the system using the packages repo (`pkg-get sysup`).
7. Reinstall necessary packages after a `revdep` check.

By default, `pkg-sync` promts at each step. Parsing the `-a, --auto` option will run all steps without prompting.

Note that `pkg-get` should be set up first!

## pkg-diff

This is also meant for systems managed by [pkg-get](https://crux.nu/portdb/?a=search&q=pkg-get) to install prebuilt packages (at some point I may merge the two tools). A `pkg-get sync` is only aware of packages with changed version/release, but will not notice those that have been rebuilt, for example after a `revdep` report for missing libraries.

The tool compares a freshly synced PKGREPO file with a previous copy, that represents the packages currently installed (or should). The `pkg-diff` script will output the differences between installed packages found in both. It will distinguish between packages with a changed version/release and those that differ only in their .md5sum, for examaple after a rebuild.

At the moment `pkg-diff` does not report new or removed packages in PKGREPO.

To create a snapshot of the current PKGREPO:

```         
pkg-diff -s
```

After a `pkg-get sync`, simply running `pkg-diff` will output a verbose report:

```         
pkg-diff

-- differences ([u] = updated, [r] = rebuilt)
[u] apr-util
[u] dhcpcd
[u] libinput
[u] libusb
[r] nss
[r] openmpi
[r] pciutils
[r] python3
[u] util-linux
```

To output only the packages that have not changed versions, but have been rebuilt, do:

```         
pkg-diff -lr

nss
openmpi
pciutils
python3
```

## prtskim

This cleans and updates port(s), by running `prtwash`, removing any `.md5sum`, and updateing footprint and signature. It can be run on a single port or a collection of ports, e.g. `prtskim *`.

## revlib

Find which ports contain missing libraries needed by a package.

Unlike `revdep`, which will tell you what packages need to be recompiled after a `prt-get sysup`, this small tool will report which ports contain missing libraries required by a certain package.

It is mainly intended for CRUX systems managed `pkg-get`. For example, I have set the packages I build on my PC as a packages repository for my laptop. I have generated the repo by `pkg-repgen` where the dependencies information is extracted from the respective ports. When I install a package on my laptop with `pkg-get depinst`, its dependencies will be pulled, however it may have linked to something more present on my build system. In this case, `pkg-get` will not be aware of such packages, that are additionally required. It is also useful when updates are done by `pkg-get sysup`: some packages may have been recompiled against newer dependencies. These would be automatically upgraded, however their dependent packages would still have the same version, and thus would be ignored in the upgrade.

Instead of running every time `revdep` and then hunting the missing libraries by `prt-get fsearch`, `revlibpkg` aims to automate things a bit.

The ports database (and hence .footprints) should be updated for this to work properly.
