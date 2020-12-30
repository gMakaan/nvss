Language: **EN**, other: [RU](README-ru.md)

> **Note:** *The publication will be after reaching the first release version - mvcs-1.0.0*

The current status of the release development completion:

![progress](https://progress-bar.dev/80/?title=1.0.0)

----

# MVCS - Manual VCS - Manual Version Control System

* MVCS - Standardization of Object Version Notation
* MVCS is the "manual" version of Git
* MVCS - Where Git Is Overkill
* MVCS - optimized for manual version handling
* MVCS - parallel work without conflicts in version names

## Brief excerpt of examples:

* `somefile v1.txt` - master-version "1" + prefix "v"
* `somefile 7.3-rc1.txt` - master-version "7.3" + tag "rc1"
* `somefile mvcs-mydesktoppc-2.3.0.txt` - master-version "2.3.0" + explicit indication of the header "mvcs" and address "mydesktoppc"
* `somefile mvcs-1.0.2-mod-1-another-2.txt` - 2 changes of the modified version of "another" relative to the modified version of "mod", which, in turn, was inherited from the master-version "1.0.2"
