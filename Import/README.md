import-macports.pl
========

This is the script I use to import a large collection of MacPorts packages into the Munki software management system. It will gather a list of all active ports, build a pkg file for each one and import that pkg into your Munki repo. The 'testing' catalog is used for all packages -- you must change the catalogs yourself.

To use this script:

 * Designate one machine as your 'build machine' and install MacPorts on it. This machine must also have Xcode and the Xcode command line utilities installed.
 * Build and install all ports that you wish to distrubute to your managed systems.
 * Mount your Munki repo on this build machine
 * Configure munkiimport on the build machine (`munkiimport --configure`) so it knows where to import your packages
 * Run `import-macports.pl` to import all active ports into your Munki repo

All imported packages include dependancy and version information. The script will also attempt to find a subset of files installed by each package and add them to an `installs` array; this helps Munki determine if a package is really installed or not -- it also helps with some troublesome packages that don't handle version numbers in a sane way.

After the import is complete, a list of package names imported will be left in the file `/tmp/import-manifest`. This file should be inserted into a real Munki manifest so that your packages are actually installed on machines.