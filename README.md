# munkiports
This is my collection of scripts to integrate MacPorts builds with munki. The Mac systems that I manage typically replace Linux systems and are used by scientists for astronomocal data processing. To ease the transition from Linux, I maintain a colection of software packages in /opt/local that mimic the collection of software typically found on Linux disributions.

These scripts build, package and import this collection of packages.


## import-macports.pl

The `Import/import-macports.pl` script is used to import an active colelction of ports into a Munki repo. See `Import/README.md` for more information on this script.