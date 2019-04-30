# Utilities
_Using FreeNAS 11.2 U3_

This file lists some useful utilities included with the FreeNAS operating system.

## Locate
The `locate` command is used to find occurrences of a string in the filesystem. I've found it useful to find all the file related to a specific service, be it configuration, executable or log files. To use it, just type
```
locate <thing-to-find>
```
On first run, it may throw an error telling you to first build an index by running another command.