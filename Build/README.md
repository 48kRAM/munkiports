# build-ports

A tool to build and install a collection of MacPorts packages from a master list.

## Master list

The ports list is a text file containing a list of "port specs", one per line. A portspec can be simply the name of port:

```
alpine
aspell
aspell-dict-en
arj
```

or can optionally include a list of variants, such as:

```git  +credential_osxkeychain  +doc  +pcre  +perl5_16  +python27```

Portspec lines can be commented out with \# (hash symbol). There is no need to list all dependencies in the ports list as MacPorts will build them for you automatically.

If you have configured default variants in `variants.conf`, be sure to negate them in the portspec when overriding the default variants.

## Usage

The `build_ports` script takes no arguments. Each run will iterate through the entire list of portspecs and install each one (`port isntall foo`) on the host system.