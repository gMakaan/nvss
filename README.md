Language: **EN**, other: [RU](README-ru.md)

> **Note:** *The publication will be after reaching the first release version - nvss-1.0.0*

The current status of the release development completion:

![progress](https://progress-bar.dev/83/?title=1.0.0)

----

# NVerSS - Nested Version Semantic String

NVSS - Standardization of Object Version Designation. In other words - an analogue of Git or Svn, but where only the string with the version itself is controlled, and not the content itself. When an object changes, its version string changes.

* Full [SemVer](https://semver.org) support.
* Support for unlimited inheritance of version modifications.
* No version conflicts when working in parallel on one object.
* Support for branches.
* The presence of an address is the domain of the owner's rights.
* Optimized for manual use.

NVSS is optimized for manual versioning, which allows this system to be used where other full-fledged version control systems do not make sense or is simply impossible. For example, parallel work / review of copies of a paper document. Or work with graphic content. Or a lot of single unrelated files, templates ...

Version inheritance makes it easier to merge changes made by other users, and also allows you to know where the change came from and its relevance.

## Brief excerpt of examples

* `somefile v1.txt` - master-version "1" + prefix "v"
* `somefile 7.3-rc1.txt` - master-version "7.3" + tag "rc1"
* `somefile nvss-mydesktoppc-2.3.0.txt` - master-version "2.3.0" + explicit indication of the header "nvss" and address "mydesktoppc"
* `somefile nvss-1.0.2-mod-1-another-2.txt` - 2 changes of the modified version of "another" relative to the modified version of "mod", which, in turn, was inherited from the master-version "1.0.2"
