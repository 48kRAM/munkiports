import-macports
========

This is the script I use to import a large collection of MacPorts packages into the Munki software management system. It will gather a list of all active ports, build a pkg file for each one and import that pkg into your Munki repo. The 'testing' catalog is used for all packages -- you must change the catalogs yourself.

## Usage
```
import-macports [options]
 
     -p|--port      Import only a specific port (default is to import all active ports)
     -m|--manifest  Add imported ports to this manifest
     -d|--debug     Print verbose debugging info
```

To use this script:

 * Designate one machine as your 'build machine' and install MacPorts on it. This machine must also have Xcode and the Xcode command line utilities installed.
 * Build and install all ports that you wish to distrubute to your managed systems (see `Build/build-ports`).
 * Configure MacPorts to build flat pkg files by adding the line `package.flat   yes` to your /opt/local/etc/macports/macports.conf file
 * Mount your Munki repo on this build machine
 * Configure munkiimport on the build machine (`munkiimport --configure`) so it knows where to import your packages
 * Run `import-macports` to import all active ports into your Munki repo

All imported packages include dependancy and version information. The script will also attempt to find a subset of files installed by each package and add them to an `installs` array; this helps Munki determine if a package is really installed or not -- it also helps with some troublesome packages that don't handle version numbers in a sane way. Also, import-macports will rename some ports which are named in a way that tends to confuse Munki (having a "-number" in the actual port name).

You can tell import-macports to automatically add newly-imported MacPorts pkgs to a manifest (using Munki's `manifestutil`) by using the `-m` option to specify a manifest. After the import is complete, a list of package names imported will also be left in the file `/tmp/import-manifest`. This file can be inserted into a real Munki manifest manually if you choose.

## Known issues

 * Importing the same port a second time using the `--port` option will result in a second copy of the package being imported into the repo. Avoid doing this. (Automatic "import all" runs will not re-import an existing port.)
