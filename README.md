# munkiports
This is my collection of scripts to integrate MacPorts builds with munki. The Mac systems that I manage typically replace Linux systems and are used by scientists for astronomocal data processing. To ease the transition from Linux, I maintain a colection of software packages in /opt/local that mimic the collection of software typically found on Linux disributions.

These scripts build, package and import this collection of packages.


## import-macports

The `Import/import-macports` script is used to import an active colelction of ports into a Munki repo. See `Import/README.md` for more information on this script.

## port-update

MacPorts has a built-in `self-update` function, but that updates the _entire_ tree. Since I tree to keep my collection "stable", sometimes I only want to update a single port. Run `Tools/port-update foo` to update the port "foo".

## build-ports

Before you can import any MacPort pkgs into Munki, you have to build them. `Build/build-ports` is a tool to build and install a collection of MacPorts packages from a listing file. The list of ports to build can also include variants to use.