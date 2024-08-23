# pgAdmin4 Desktop package for Arch Linux

## What

These package sources can be used to install [pgAdmin 4](https://www.pgadmin.org/) on Arch Linux.

Functionally, the main difference to pgAdmin 4 as published in the community repo
is that, instead of providing a tray icon app with the server running in the background,
the desktop deployment is done as
[described in the pgAdmin docs](https://www.pgadmin.org/docs/pgadmin4/latest/desktop_deployment.html),
using Electron.

Practically this means that pgAdmin is shown in a Chromium-based dedicated browser
window. The web application is served by a webserver in the background as long as
the main window is open.

Earlier versions attempted to maintain compatibility with packaged Python
libaries from the community repository. This became impossible to maintain however,
for two reasons:

1. Conflicts in required versions of pgAdmin4 vs. the Arch packaged libraries increased.
   In the beginning, there were just a few patches required to make it work. Over
   time the incompatibilities grew.
2. Growing number of packages used by pgAdmin4, which were not covered by community
   or [Arch User Repository](https://aur.archlinux.org/) packages.

Therefore this packages the virtual environment along with the installation. Be aware
this adds bloat to the resulting package, but on the other hand will save you the
trouble of installing a lot of Python packages system-wide that are potentially
only needed for pgAdmin.

## How

Clone this repository, build and install:

```sh
git clone https://github.com/merll/pgadmin4-nw.git
cd pgadmin4-nw
makepkg -i
```

> Note that some parts of building the JavaScript sources can be quite heavy on memory
> usage. Try not to have too many other applications running at build time.

## Why

The existing package in the community repository of Arch Linux has been outdated for a
while. Python libraries that pgAdmin 4 depends on are used system-wide in Arch Linux,
but it has been difficult to update pgAdmin compatibility at the same pace. Mostly 
package upgrades are carried out frequently on Arch Linux, while pgAdmin still depends
on older versions (partially due to compatibility with more conservative Linux
distributions). As a consequence, package upgrades in Arch Linux often result in
pgAdmin no longer being able to start. In addition, the amount of dependencies
(i.e. Python packages) has grown beyond what is maintained in the Arch
community repository.

Some similar packages as this one had been published in the Arch User
Repository. However, all of them have been removed in the meantime due to conflicts with
[AUR Submission Rules](https://wiki.archlinux.org/title/AUR_submission_guidelines).
Recently `pgadmin4-desktop` and `pgadmin4-server` were added again, which are based on
the pgAdmin4 Debian packages distributed by the pgAdmin4 development team. Using these
may result in a larger installation size, but will be much faster because it already
comes with all Python packages and Node.js modules pre-packaged. Hopefully these stay
around in the AUR. For the time being, this repo can be exepected to be updated with
new pgAdmin4 releases as well.

## Alternatives

pgAdmin can also be installed

* [using the recently-added AUR package](https://aur.archlinux.org/packages/pgadmin4-desktop)
* [using a container](https://www.pgadmin.org/docs/pgadmin4/latest/container_deployment.html)
* [via a Python package](https://www.pgadmin.org/download/pgadmin-4-python/)
