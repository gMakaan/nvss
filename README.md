# NVerSS - Nested Version Semantic String - Semantic version string with inheritance

Current NVSS revision: **nvss-1.0.0-en**

NVSS - Standardization of object versioning. A pocket analogue of Git or Svn, where only the version string itself is controlled, not the content itself. When an object changes, its version string changes in accordance with NVSS rules.

* **No version conflicts when working on copies of the same object in parallel**.
* Full support for [SemVer](https://semver.org).
* Support for unlimited inheritance of version modifications.
* Support for branches.
* Availability of addresses and scope of rights for authors of versions.
* Human readable syntax. Adapted for handmade.

NVSS is optimized for manual versioning, which allows you to use this system where full-fledged version control systems are redundant or simply impossible. For example, parallel work/review of printed documents. Or work with graphic content. Or many single unrelated files, templates, documents...

Version inheritance makes it easier to combine different modifications, and also allows you to find out where the modification came from and its relevance.

The simplicity of the syntax allows you to quickly make parsers for automation.

## Contents

- [NVerSS - Nested Version Semantic String - Semantic version string with inheritance](#nverss---nested-version-semantic-string---semantic-version-string-with-inheritance)
  - [Contents](#contents)
  - [Quick reference](#quick-reference)
  - [Definitions](#definitions)
  - [Comparison with git](#comparison-with-git)
  - [Signs of using NVSS with an object](#signs-of-using-nvss-with-an-object)
  - [Master-version of the object](#master-version-of-the-object)
    - [Format of the master-version of the object](#format-of-the-master-version-of-the-object)
    - [Description of the structure of the master-version of the object](#description-of-the-structure-of-the-master-version-of-the-object)
  - [Change object](#change-object)
    - [Format of modifications and branches](#format-of-modifications-and-branches)
    - [Description of the structure of modifications and branches](#description-of-the-structure-of-modifications-and-branches)
  - [Syntax](#syntax)
    - [Delimiters](#delimiters)
    - [Allowed characters in prefixes, addresses and names](#allowed-characters-in-prefixes-addresses-and-names)
  - [Description of the main actions with examples](#description-of-the-main-actions-with-examples)
    - [Master version: create `(init)`](#master-version-create-init)
    - [Master version: change `([fetch downstream + merge] | commit)`](#master-version-change-fetch-downstream--merge--commit)
    - [Master version: change of address](#master-version-change-of-address)
    - [Master version: fix `(amend)`](#master-version-fix-amend)
    - [Modification: creation `(clone upstream + commit)`](#modification-creation-clone-upstream--commit)
    - [Modification: change `([fetch downstream + merge] | commit)`](#modification-change-fetch-downstream--merge--commit)
    - [Modification: fix `(amend)`](#modification-fix-amend)
    - [Modification: base change `(pull upstream)`](#modification-base-change-pull-upstream)
    - [Modification: address change](#modification-address-change)
    - [Branch: creation `(branch [+ commit])`](#branch-creation-branch--commit)
    - [Branch: change `(commit)`](#branch-change-commit)
    - [Branch: fix `(amend)`](#branch-fix-amend)
    - [Branch: base change `(merge + commit)`](#branch-base-change-merge--commit)
    - [Branch: change name `(branch -m)`](#branch-change-name-branch--m)
  - [NVSS examples](#nvss-examples)
  - [Links and extra info](#links-and-extra-info)

## Quick reference

Master-version of the object:

```ebnf
[nvss "-"] [masterAddress "-"] [verPrefix] (version | timeSnapshot).
```

Object modification:

```ebnf
masterBase "-" {nestedBase "-"} modAddress "-" modChanges.
```

Object branch:

```ebnf
masterBase "-" {nestedBase "-"} "." branchName "-" branchChanges.
```

Full version:

```ebnf
master {mod | branch}
```

## Definitions

* NVSS - Nested Version Semantic String
* SemVer - semantic versioning (see [SemVer](https://semver.org))
* object - any software or physical object that requires the use of version control
* master-version - major/root version of an object, assigned by the object's author
* modification - a modified version of the object with a different address
* branch - a modified version of the object without changing the address (and editing rights)
* base - the original (inherited) version of the object (and part of the NVSS string) on which the modification or branch is based
* address - defines (and distinguishes) the location of the object's version
* author - creator of the master-version, modification, or branch

Definitions - actions

* change - action after which the version of the object must be changed
* fix - an action after which the version of the object does not change. It is allowed to use if this version is not used anywhere
* merge - the process of merging different versions of an object

## Comparison with git

|   git    | NVSS                                                                                                          |
| :------: | :------------------------------------------------------------------------------------------------------------ |
|   init   | assigning a master-version to an object                                                                       |
|  clone   | simple object copying and modification creation                                                               |
|  fetch   | object versions from different authors are accepted                                                           |
|   pull   | versions of the object from different authors are accepted, merged and the version is changed                 |
|   push   | the current version of the object is sent                                                                     |
| branches | simple object copy and branch creation                                                                        |
|  merge   | merging different versions of an object                                                                       |
|  commit  | version string change                                                                                         |
|  amend   | object correction                                                                                             |
|   add    | adding a new object to a group when controlling several objects under one version                             |
|  remove  | deleting an object from a group when controlling several objects under one version                            |
|  revert  | if the required version of the object exists, deleting the current one and replacing it with the required one |

## Signs of using NVSS with an object

* The presence of the `nvss-` prefix in the object version (explicit use).
* The version of the object complies with the current NVSS versioning rules (implicit usage).

## Master-version of the object

The master-version is the main version of the object.

Version control of an object in NVSS format must begin with assigning a master-version to it. The version string can be merged with anything (eg object name) using any available delimiter. The beginning of the version string is recommended to be determined by the `nvss` keyword or by the presence of the *master-version* address and the `-` separator before the version designation (see below).

* **Guarantees no third-party modifications** when used.
* Implicitly the main branch of an item, and in most cases the main working version of an item.

### Format of the master-version of the object

```ebnf
[nvss "-"] [masterAddress "-"] [verPrefix] (version | timeSnapshot).
```

> **Note:** *The `[]` brackets are optional parts.*

### Description of the structure of the master-version of the object

```ebnf
["nvss"]
```

Optional `nvss` header tag with `-` delimiter.

* Recommended to explicitly start the version string in NVSS format.
* Can be used as a search tag.
* May be omitted if there is version information in the object name or known to use NVSS with the object to shorten the length of the version string.

```ebnf
[masterAddress]
```

Optional **unique** address of the master-version of the object, delimited by `-`.

* This address identifies the author of the master-version.
* Must be clear and concise so that it can be quickly and accurately identified.
* May be absent if it is known in advance or unmistakably and easily determined by some other means or features. It may also be absent if there is uncertainty in the wizard.
* May change, appear and disappear as needed, while it is desirable to notify other dependent authors in advance to maintain their connection. It is recommended to assign a new address with a mention of the old address (double address) in it for a while, to remind others of the change that has taken place.

`masterAddress` **should always include at least one letter** to be uniquely identified in the string, starting with a letter is recommended.

```ebnf
[verPrefix]
```

An optional prefix to the version designation. Eg. `v`, `rev` or whatever.

Recommended for use with a short version string, with no `nvss` header tag and no `masterAddress` address of the master-version, for a more "distinguishable" version designation.

Often used:

* `v`, `ver` - version
* `r`, `rev` - revision
* `i`, `itr` - iteration
* `s`, `snp` - snapshot
* `b`, `bak` - backup copy (backup)

```ebnf
version
```

Designation *master-version*.

* Changes every time the object changes.
* Does not change when changing `masterAddress`.
* It is possible to fix the object (combine the object without changing the version **with the guaranteed absence of this version being used anywhere**).

The version designation structure uses [semantic versioning](https://semver.org) for the `major`, `minor`, `patch` parts and allows several formats (in order to simplify):

* `major "." minor "." patch ["-" tag {"." tag}]` - semver-compatible (`tag` is similar to `pre-release` with some restrictions (see description of `tag` below) )
* `major "." minor ["-" tag {"." tag}]`
* `major ["-" tag {"." tag}]`

or in general:

```ebnf
version = major ["." minor | ("." minor "." patch)] ["-" tag {"." tag}].
```

Description and characteristics of the fields:

* `major` - major version (number), incremented on major changes (causing incompatibility with previous `major` version)
* `minor` - minor version (number), increases with minor changes (there is compatibility with the current `major` version)
* `patch` - patch of the current version (number), incremented by bug fixes, patches (there is compatibility with the current `major.minor` version)
* `tag` - optional tags of the current version separated by `.`. Unlike semver, NVSS disallows usage of the `-` character in tag names. Used to explain the features of a version, e.g. `-rc1` or `-dev` or `-prerelease` etc.

> **Note:** *The `{}` brackets contain parts repeated 0 or more times.*

```ebnf
time Snapshot
```

Timestamp (timecode) of the master-version.

* Changes every time the object changes.
* Does not change when changing `masterAddress`.
* It may not necessarily change to the current time, but also to the past, if it is later than indicated in the version.
* It is possible to fix the object (combine the object without changing the version **with the guaranteed absence of this version being used anywhere**).

The timestamp structure must conform to [ISO 8601](https://ru.wikipedia.org/wiki/ISO_8601) and allows multiple formats. Valid time format masks (in order of simplification):

* `"YYYYMMDDThhmmss.sss" [utcOffset]`
* `"YYYYMMDDThhmmss" [utcOffset]`
* `"YYYYMMDDThhmm" [utcOffset]`
* `"YYYYMMDDThh" [utcOffset]`
* `"YYYYMMDD" [utcOffset]`
* `"YYYY" [utcOffset]`

> **Note:** *`"YYYYMM" [utcOffset]` is not supported by ISO 8601 (conflict with `"YYMMDD"`)*

Abbreviated formats with 2-digit year

* `"YYMMDDThhmmss.sss" [utcOffset]`
* `"YYMMDDThhmmss" [utcOffset]`
* `"YYMMDDThhmm" [utcOffset]`
* `"YYMMDDThh" [utcOffset]`
* `"YYMMDD" [utcOffset]`
* `"YY" [utcOffset]`

> **Note:** *`"YYMM" [utcOffset]` conflicts with `"YYYYMM"`)*

Description of the mask:

* `YYYY` - year (4-digit format)
* `YY` - year (2-digit abbreviated format)
* `MM` - month (2-digit format, with leading zeros)
* `DD` - day (2-digit format, with leading zeros)
* `T` - date-time separator (2-digit format, with leading zeros)
* `hh` - hours (2-digit format, with leading zeros)
* `mm` - minutes (2-digit format, with leading zeros)
* `ss` - seconds (2-digit format, with leading zeros)
* `.` - dot, separating seconds from milliseconds
* `sss` - milliseconds (3-digit format, with leading zeros)
* `utcOffset` - offset from UTC (letter)

By default, without `utcOffset`, the mask specifies **local** time. To use global time, UTC is added to the end of the `Z` tag and the time is specified as **global**. Explicit local time

The [military time zones](https://en.wikipedia.org/wiki/List_of_military_time_zones) (letter designation) can be used to indicate an offset from UTC:

| A  | b  | c  | D  | e  | F  | G  | h  | I  | K   | l   | M   |
| -- | -- | -- | -- | -- | -- | -- | -- | -- | --- | --- | --- |
| +1 | +2 | +3 | +4 | +5 | +6 | +7 | +8 | +9 | +10 | +11 | +12 |

Positive offset

| Z |
| - |
| 0 |

UTC - no offset

| N  | O  | P  | Q  | R  | S  | T  | U  | v  | w   | x   | Y   |
| -- | -- | -- | -- | -- | -- | -- | -- | -- | --- | --- | --- |
| -1 | -2 | -3 | -4 | -5 | -6 | -7 | -8 | -9 | -10 | -11 | -12 |

Negative offset

The letter J is the local time (equivalent to the absence of the `utcOffset` time zone letter).

> **Note:** *Time zones (for global time offset) in numeric format are not supported in NVSS because include the forbidden character `+`.*

```ebnf
timeSnapshot = YY MM DD ["T" hh [mm [ss ["." sss]]]] [utcOffset].
```

## Change object

Modifying **copies** of the base object under NVSS control.

There are 2 types of object modification:

* *Modifications* - version of the object with a new unique address. Modification rights are transferred to this address
* *Branches* - the object version is always located at the same address as the branch base (does not change), the branch name is prefixed with `.` (dot). The period is always implicitly preceded by the address of the underlying object. The rights to modify are not transferred, the author of the branch is determined by the database.

> **Note:** *If you need to immediately create a branch at a new address, you must first create an empty modification (zero number of changes) to set the address.*

if the object does not change (exporting a piece of a.svg to a separate another a.svg) then this is a pure version of the branch
otherwise, you can inherit the version string from the main object and immediately create a branch with a zero change and a name, eg. export -->

Versions should be changed mostly in the reverse order along the entire chain - passing the modification / branch to the base. This ensures that the versions as a whole are up-to-date and that it is easier to compare different versions for later merging.

> **Attention:** *In case of address conflicts or suspected address substitution, you must obtain up-to-date information from the author at this address*

### Format of modifications and branches

```ebnf
masterBase "-" {nestedBase "-"} modAddress "-" modChanges.
masterBase "-" {nestedBase "-"} "." branchName "-" branchChanges.
```

Unlimited multiple inheritance is available.

With multiple inheritance, to reduce the length of the version string, you can use a shorthand option - hide any parts of internal bases through ellipsis `...`, but **the master-version and the nearest base must always be present** to maintain a connection with the base version:

```ebnf
masterBase "-" {{"...-"} nestedBase "-"} modAddress "-" modChanges.
masterBase "-" {{"...-"} nestedBase "-"} "." branchName "-" branchChanges.
```

In this case, you can learn about hidden bases only from the nearest base, where they will be revealed in its version line.

> **Danger:** *When bases are hidden, the outermost visible bases may look the same, but depend on different intermediate bases. Therefore, merging a master-version with a version with hidden bases is highly discouraged (only possible if intermediate bases are unavailable, which must be eliminated later).*

> **Recommendation:** *Multiple inheritance of modifications/branches with a large number of bases should be avoided, especially in conjunction with hidden bases, because this slows down and complicates the merge procedure, requires checking the current versions of all databases. The length of the version string is also increased.*

### Description of the structure of modifications and branches

```ebnf
masterBase
```

The master-version of the object from which the change is made. **Editing this part of the line is prohibited**.

* Contains the full master-version string in the format described above, with a `-` delimiter at the end.

```ebnf
{nestedBase}
```

The next modification or branch of the object from which the change is made. **Editing this part of the line is prohibited**.

* Contains the full string of the base version in the format described above, with a `-` delimiter at the end.
* The number of successive inherited object modifications is unlimited.

```ebnf
modAddress
branchName
```

**Unique mandatory** modification address of `modAddress` object with `-` separator or **unique mandatory** branch name of `branchName` object with `.` prefix and `-` separator.

* Assigned at the first modification of the basic version.
* The author of the modification is determined by the address.
* The branch author is the master-version or extreme base modification, ignoring possible intermediate inherited branches.
* Subject to change, it is desirable to notify in advance dependent authors of this modification/branch in order to maintain their connection.

`modAddress` and `branchName` **must always include at least one letter** to be uniquely identified in a string, starting with a letter is recommended.

```ebnf
modChanges
branchChanges
```

The number of changes to the `modChanges` modification or `branchChanges` branch is an integer.

* Each time the object is changed, it increases by 1.
* May start from 0 if not modified to assign an address/name in advance.
* Indicates the lead (number of changes) relative to the base version of `base`.
* It is possible to fix the object (merging the object without increasing the number of changes **with the guaranteed absence of this version being used anywhere**).

## Syntax

### Delimiters

* **`.`** - version delimiter, prefix before branch names
* **`-`** - separator of NVSS string elements (addresses/names/tags/versions), branch

> **Note:** *Standard characters, present in all encodings, allowed in filenames of all current operating systems. If the use of these symbols is prohibited, the use of non-standard ones is allowed, but an additional explanation is required for their use, and as soon as possible, their conversion to standard ones.*

### Allowed characters in prefixes, addresses and names

Hard mode (with limited character set)

* Arabic numerals
* lowercase (if available) English letters
* underscore character `_` to separate words. Between two numbers and several in a row is prohibited. If a symbol is not available, join words together

For normal use

* numbers of any language supported by the target ecosystem (not causing problems when used)
* lowercase (if available) letters of any language supported by the target ecosystem (not causing problems when used)
* underscore character `_` to separate words. Between two numbers is not recommended. Several in a row prohibited

## Description of the main actions with examples

### Master version: create `(init)`

* adding a master-version line to comply with NVSS rules. The version designation must comply with SemVer's semantic versioning rules.

| before |    | after |
| ------ | -- | ----- |
|        | →  | v1.0  |

### Master version: change `([fetch downstream + merge] | commit)`

* change `version` or `timeSnapshot` according to NVSS rules.

| before |    | after |
| ------ | -- | ----- |
| v1.0   | →  | v2.0  |

### Master version: change of address

* changes (`masterAddress`) to another. The designation or timestamp of the master-version does not change.

> **Recommendation:** *it is recommended to assign a new address with a mention of the old address (double address) in it for a while, to remind other authors about the change.*

| before (options) |    | after (options)           |
| ---------------- | -- | ------------------------- |
| v1.0             | →  | creator-v1.0              |
| creator-v1.0     | →  | altername-v1.0            |
| creator-v2.0     | →  | altername-v2.0            |
| creator-v2.0     | →  | creator_to_altername-v2.0 |
| creator-v2.0     | →  | creator2altername-v2.0    |

### Master version: fix `(amend)`

> **Note:** *available only if it is guaranteed that the current master-version of the object is not used anywhere.*

* `version` or `timeSnapshot` is not incremented, this counts as a fix to the current version.

| before |    | after |
| ------ | -- | ----- |
| v1.0   | →  | v1.0  |

### Modification: creation `(clone upstream + commit)`

* when making changes to an object, add `modAddress` and set `modChanges` to 1;
* if there are no changes to the object, add `modAddress` and set `modChanges` to 0. Used to assign a modification address in advance, before changes.

| before (options) |    | after (options) |
| ---------------- | -- | --------------- |
| v1.0             | →  | v1.0-mod-1      |
| v1.0             | →  | v1.0-mod-0      |

### Modification: change `([fetch downstream + merge] | commit)`

* increase `modChanges` according to NVSS rules.

| before     |    | after      |
| ---------- | -- | ---------- |
| v1.0-mod-1 | →  | v1.0-mod-2 |

### Modification: fix `(amend)`

> **Attention:** *only available if it is guaranteed that this version of the modification of the object is not used anywhere.*

* `modChanges` is not incremented, this is considered a fix to the current version.

| before     |    | after      |
| ---------- | -- | ---------- |
| v1.0-mod-1 | →  | v1.0-mod-1 |

### Modification: base change `(pull upstream)`

> **Note:** *The current database may become unavailable for some reason, out of date or change address - then the new database will be with a different address. Use it after making sure it is up-to-date and compatible with the current modification.*

* the new database is combined with the current version of the modification;
* in the absence of differences from the base after the merge - `modChanges` is set to 0. Used to save the address of the modification, before the changes;
* if there are differences from the base after the merge - `modChanges` is reset to 1.

| before (options) |    | after (options) |
| ---------------- | -- | --------------- |
| v1.0-mod-1       | →  | v2.0-mod-0      |
| v1.0-mod-1       | →  | v2.0-mod-1      |

### Modification: address change

* `modAddress` changes, while `modChanges` may not be reset to 1.

| before (options) |    | after (options) |
| ---------------- | -- | --------------- |
| v1.0-mod-1       | →  | v1.0-m-1        |
| v1.0-mod-2       | →  | v1.0-m-2        |

### Branch: creation `(branch [+ commit])`

* when making changes to an object, add `branchName` and set `branchChanges` to 1;
* if there is no change to the object, add `branchName` and set `branchChanges` to 0. Used to assign a branch name ahead of time before changes are made.

| before (options) |    | after (options) |
| ---------------- | -- | --------------- |
| v1.0             | →  | v1.0-.dev-1     |
| v1.0             | →  | v1.0-.dev-0     |

### Branch: change `(commit)`

* increase `branchChanges` according to NVSS rules.

| before      |    | after       |
| ----------- | -- | ----------- |
| v1.0-.dev-1 | →  | v1.0-.dev-2 |

### Branch: fix `(amend)`

> **Note:** *only available if it is guaranteed that this version of the object branch is not used anywhere.*

* `branchChanges` is not incremented, this counts as a fix to the current version.

| before      |    | after       |
| ----------- | -- | ----------- |
| v1.0-.dev-1 | →  | v1.0-.dev-1 |

### Branch: base change `(merge + commit)`

* the new database is merged with the current version of the branch;
* if there are no differences from the base after the merge - `branchChanges` is set to 0. Used to save the branch address before changes;
* if there are differences from the base after merging - `branchChanges` is reset to 1.

> **Note:** *Branching is only available within the scope of the address of the current modification or master-version.*

| before (options)  |    | after (options)   |
| ----------------- | -- | ----------------- |
| v1.0-mod-1-.dev-1 | →  | v1.0-mod-2-.dev-0 |
| v1.0-mod-1-.dev-1 | →  | v1.0-mod-2-.dev-1 |
| v1.0-.dev-1       | →  | v2.0-.dev-1       |

### Branch: change name `(branch -m)`

* `branchName` changes, while `branchChanges` may not be reset to 1.

| before          |    | after       |
| --------------- | -- | ----------- |
| v1.0-.develop-2 | →  | v1.0-.dev-2 |

## NVSS examples

* `1` - master-version "1"
* `v1` - master-version "1" + prefix "v"
* `7.3` - master-version "7.3"
* `rev7.3` - master-version "7.3" + "rev" prefix
* `0.7.3` - master-version "0.7.3"
* `7.3-rc1` - master-version "7.3" + tag "rc1"
* `mydesktoppc-0.7.3` - master-version "0.7.3" + address "mydesktoppc"
* `nvss-0.7.3` - master-version "0.7.3" + explicit header "nvss"
* `nvss-mydesktoppc-2.3.0` - master-version "2.3.0" + explicit header "nvss" and address "mydesktoppc"
* `nvss-1.0.0-mod.1` - master-version "1.0.0" + tags "mod" and "1"
* `nvss-1.0.0-mod-1` - 1 change of modification "mod" from master-version "1.0.0"
* `nvss-1.0.0-.mod-1` - 1 change of branch "mod" from master-version "1.0.0"
* `nvss-1.0.2-mod-1-another-2` - 2 changes of modification "another" relative to modification "mod", which in turn was inherited from master-version "1.0.2"

Using the example file `somefile`:

* `somefile 7.3.txt` - master-version "7.3"
* `somefile 7.3-rc1.txt` - master-version "7.3" + tag "rc1"
* `somefile rev7.3.txt` - master-version "7.3" + "rev" prefix
* `somefile nvss-rev7.3.txt` - master-version "7.3" + prefix "rev" + explicit header "nvss"

Process example 1:

1. Master version of the object: `nvss-v3`
2. Author 1 got the object and created his modification: `nvss-v3-user1-1`
3. Author 2 received a modification of the object from author 1 and also made his own changes: `nvss-v3-user1-1-user2-1`
4. The master-version of the object has changed in the meantime: `nvss-v5`
5. Both authors returned their modifications to the author of the master-version. Now the master author needs to merge their changes with his `nvss-v5` version and release `nvss-v6`. The author of the master-version can see that authors 1 and 2 edited the version of `nvss-v3` and that the version of author 2 includes the changes of author 1 and is newer (because it inherits from the base version of `nvss-v3-user1-1`) . This will help you quickly understand the changes made and change your master-version.

Process example 2:

1. An object without a version is received. Options for action:
   * Don't use versioning until the original author of the object assigns a version.
   * Put your own master-version in the form of a timestamp without explicitly specifying the address of the master-version.
   * Assign a full-fledged master-version by prior arrangement with the original author of the object. He will continue to use it.
2. After setting the master-version for the original author, you can fully work with the object in accordance with the rules of NVSS.

## Links and extra info

* [Extended Backus–Naur form](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_form)
* [Semantic Versioning](https://semver.org)
* [Difference between version and revision terms](https://www.product-lifecycle-management.com/plm-revision-version.htm)
* [File Naming for WEB](https://www.mtu.edu/umc/services/digital/writing/characters-avoid/)
* [Features of using semver](https://developer.okta.com/blog/2019/12/16/semantic-versioning)
