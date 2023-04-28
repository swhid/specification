# 5 Core identifiers

A core SWHID identifier is composed of four fields, separated by a colon
`:`.  The `swh` prefix makes it explicit that this is a SWHID identifier.  `1`
(`<scheme_version>`) is the current version of this identifier *scheme*. Future
editions will use higher version numbers, possibly breaking backward
compatibility. The third field is a tag that correspond to the type of object
identified:

-   `cnt` for **contents**.
-   `dir` for **directories**,
-   `rev` for **revisions**,
-   `rel` for **releases**,
-   `snp` for **snapshots**,

The fourth and last field is the *intrinsic identifier* of the object.  In this
version of the specification, this is a hex-encoded (using lowercase ASCII
characters) SHA1 computed on the content and relevant metadata of the object
itself, as follows.

## 5.1 Contents

A *content* is an uninterpreted byte sequence, typically, the content of a file.
For this type of object the intrinsic identifier is the `sha1_git` hash of it,
i.e. the SHA1 of the byte sequence obtained by juxtaposing

 - the ASCII string `"blob"` (without quotes),
 - an ASCII space,
 - the length of the content as ASCII-encoded decimal digits,
 - a NULL byte,
 - and the actual content of the file.

No metadata is used for this type of object (in particular, notice that there
is no file *name* mentioned here).

As an example, `swh:1:cnt:94a9ed024d3859793618152ea559a168bbcbb5e2` is the SWHID computed from [the full text of the GPL3 license](https://archive.softwareheritage.org/swh:1:cnt:94a9ed024d3859793618152ea559a168bbcbb5e2)

## 5.2 Directories ##

Directories are data structures commonly used in hierarchical file systems to group together files and other directories, and to hold relevant metadata about them, in the form of directory entries.

This version of the SWHID standard adopts the same convention as the popular *git* version control system, and only takes into account as metadata the name of the directory entries (as a sequence of arbitrary bytes, excluding ASCII '/' and the NULL byte) and a simplified representation of the access rights.

The names of entries in a directory must be distinct from one another.

In order to compute the intrinsic identifier of a directory, it is necessary to compute first the SWHID of each object listed in the directory.

Then one proceeds to create a serialization of the directory as follows:

 1. sort the directory entries using the following algorithm
     1. for each entry pointing to a *directory*, append an ASCII '/' to its name
     2. sort all entries using the natural byte order of their (modified) name

 2. for each entry, with a given *name* (unmodified), add a sequence of bytes composed of
     1. the normalized access rights, encoded as a sequence of ASCII-encoded octal digits ('100644' for regular files, '100755' for executable files, '120000' for symbolic links, '40000' for directories),
     2. an ASCII space,
     3. the *name* as a raw string of bytes
     4. a NULL byte
     5. the *intrinsic identifier* of the *content* or *directory*, encoded as a sequence of 20 bytes

The intrinsic identifier of the directory is the SHA1 of the byte sequence obtained by juxtaposing

 - the ASCII string `"tree"` (without quotes),
 - an ASCII space,
 - the length of the previously obtained serialization as ASCII-encoded decimal digits,
 - a NULL byte,
 - and the previously obtained serialization.

As an example, `swh:1:dir:d198bc9d7a6bcf6db04f476d29314f157507d505` is the
is the SWHID computed from
[a directory containing the source code of the darktable photography application](https://archive.softwareheritage.org/swh:1:dir:d198bc9d7a6bcf6db04f476d29314f157507d505) as
a given point in time of its development on May 4th 2017.

## 5.3 Revisions

Software development within a specific project is essentially a time-indexed series of copies of a single “root” directory that contains the entire project source code. Software evolves when a developer modifies the content of one or more files in that directory and record their changes.

Each recorded copy of the root directory is known as a “revision”. It points to a single fully-determined directory and is equipped with arbitrary metadata. Some of those are added manually by the developer (e.g., revision message), others are automatically synthesized (timestamps, parent revision(s), etc).

The supported metadata is as follows:

 - author (arbitrary byte sequence, mandatory): generally contains the name and email address of the author of the revision.
 - author timestamp (decimal timestamp from the Unix epoch, mandatory): the date at which the revision was authored.
 - author timezone offset (arbitrary byte sequence): UTC offset at which the revision was authored, usually an ASCII-encoded [+/-]HHMM specification.
 - committer (arbitrary byte sequence, mandatory): generally contains the name and email address of the committer of the revision.
 - committer timestamp (decimal timestamp from the Unix epoch, mandatory): the date at which the revision was committed.
 - committer timezone offset (arbitrary byte sequence): UTC offset at which the revision was committed, usually an ASCII-encoded [+/-]HHMM specification.
 - directory (mandatory): the root directory recorded by the revision
 - parent revisions (ordered list of revisions): the immediately preceding revisions in the development timeline. Can be empty for an initial revision, and have multiple revisions when multiple branches of history are being merged.
 - extra headers (ordered list of byte key/value pairs): arbitrary additional metadata attached to the revision (FIXME: add examples, add constraints)
 - message: the message describing the revision

In order to compute the intrinsic identifier of a revision, it is necessary to first compute the intrinsic identifier of the root directory recorded by the revision, as well as the intrinsic identifier of all parent revisions (recursively).

The serialization of the revision is a sequence of lines in the following order:

 - the reference to the root directory:
    - the ASCII string `"tree"` (4 bytes)
    - an ASCII space
    - the ASCII-encoded hexadecimal intrinsic identifier of the directory (40 ASCII bytes)
    - a LF
 - for each parent revision, in the order they've been provided, a reference to that revision:
    - the ASCII string `"parent"` (6 bytes)
    - an ASCII space
    - the ASCII-encoded hexadecimal intrinsic identifier of the parent revision (40 ASCII bytes)
    - a LF
 - the author line:
    - the ASCII string `"author"` (6 bytes)
    - an ASCII space
    - the string of bytes provided for the author name and email, with each LF replaced by LF followed by an ASCII space
    - an ASCII space
    - the ASCII-encoded decimal representation of the author timestamp
    - an ASCII space
    - the string of bytes provided for the author timezone offset, with each LF replaced by LF followed by an ASCII space
    - a LF
 - the committer line:
    - the ASCII string `"committer"` (9 bytes)
    - an ASCII space
    - the string of bytes provided for the committer name and email, with each LF replaced by LF followed by an ASCII space
    - an ASCII space
    - the ASCII-encoded decimal representation of the committer timestamp
    - an ASCII space
    - the string of bytes provided for the committer timezone offset, with each LF replaced by LF followed by an ASCII space
    - a LF
 - the extra header lines; for each provided key/value pair, in the order they have been provided:
    - the key
    - an ASCII space
    - the value, with each LF replaced by LF followed by an ASCII space
    - a LF
 - if the message is defined:
    - an extra LF (the message is separated from the header with two LFs)
    - the commit message as a raw string of bytes

The intrinsic identifier of the revision is the SHA1 of the byte sequence obtained by juxtaposing

 - the ASCII string `"commit"` (without quotes),
 - an ASCII space,
 - the length of the previously obtained serialization as ASCII-encoded decimal digits,
 - a NULL byte,
 - and the previously obtained serialization.

As an example, `swh:1:rev:309cf2674ee7a0749978cf8265ab91a60aea0f7d` is the SWHID computed from
[a commit in the development history of Darktable](https://archive.softwareheritage.org/swh:1:rev:309cf2674ee7a0749978cf8265ab91a60aea0f7d), dated 16 January 2017, that added undo/redo supports for masks.


## 5.4 Releases

Some revisions get selected by developers as denoting important project milestones known as “releases”. Each release points to the last commit in project history corresponding to the release and carries metadata: release name and version, release message, cryptographic signatures, etc. If they're not attached to development history (e.g. if they've been imported from bare tarballs), releases can also point directly to a root directory instead of a full revision with metadata.

The supported metadata is as follows:
 - name (arbitrary byte sequence, mandatory): a name identifying the release
 - author (arbitrary byte sequence): generally contains the name and email address of the author of the release.
 - author timestamp (decimal timestamp from the Unix epoch): the date at which the release was authored.
 - author timezone offset (arbitrary byte sequence): UTC offset at which the release was authored, usually an ASCII-encoded [+/-]HHMM specification.
 - target object (mandatory): a reference to another object, which can be either a revision, a directory or less commonly a content or another release
 - message: the message describing the release

In order to compute the intrinsic identifier of a release, it is necessary to first compute the intrinsic identifier of the targeted object.

The serialization of the release is a sequence of lines in the following order:

 - the reference to the target object:
    - the ASCII string `"object"` (6 bytes)
    - an ASCII space
    - the ASCII-encoded hexadecimal intrinsic identifier of the target object (40 ASCII bytes)
    - a LF
    - the ASCII string `"type"` (4 bytes)
    - an ASCII space
    - an ASCII string referencing the type of the target object (`"commit"` for a revision, `"tree"` for a directory, `"tag"` for another release, `"blob"` for a content object)
    - a LF
 - the name of the release:
    - the ASCII string `"tag"` (3 bytes)
    - an ASCII space
    - the string of bytes provided for the release name, with each LF replaced by LF followed by an ASCII space
    - a LF
 - if there is an author, the author line:
    - the ASCII string `"tagger"` (6 bytes)
    - an ASCII space
    - the string of bytes provided for the author name and email, with each LF replaced by LF followed by an ASCII space
    - an ASCII space
    - the ASCII-encoded decimal representation of the author timestamp
    - an ASCII space
    - the string of bytes provided for the author timezone offset, with each LF replaced by LF followed by an ASCII space
    - a LF
 - if the message is defined:
    - an extra LF (the message is separated from the header with two LFs)
    - the commit message as a raw string of bytes

The intrinsic identifier of the release is the SHA1 of the byte sequence obtained by juxtaposing

 - the ASCII string `"tag"` (without quotes),
 - an ASCII space,
 - the length of the previously obtained serialization as ASCII-encoded decimal digits,
 - a NULL byte,
 - and the previously obtained serialization.

As an example, `swh:1:rel:22ece559cc7cc2364edc5e5593d63ae8bd229f9f` is the SWHID computed from
the [Darktable release 2.3.0](https://archive.softwareheritage.org/swh:1:rel:22ece559cc7cc2364edc5e5593d63ae8bd229f9f), dated 24 December 2016.


## 5.5 Snapshots

Any kind of software origin offers multiple pointers to the “current” state of a development project. In the case of VCS this is reflected by branches (e.g., master, development, but also so called feature branches dedicated to extending the software in a specific direction); in the case of package distributions by notions such as suites that correspond to different maturity levels of individual packages (e.g., stable, development, etc.).

A “snapshot” of a given software origin records all entry points found there and where each of them was pointing at the time. For example, a snapshot object might track the commit where the master branch was pointing to at any given time, as well as the most recent release of a given package in the stable suite of a FOSS distribution.

Practically, a snapshot is a list of named branches pointing at objects of any of the known types (content, directory, revision, release or snapshot). A branch can also be an alias to another (named) branch, for instance the default `"HEAD"` branch can point at another, more specific, `"refs/heads/main"` branch.

To compute the intrinsic identifier of a snapshot, one must first compute the intrinsic identifier of all objects referenced by the snapshot.

Then one proceeds to create a serialization of the snapshot as follows:

 1. sort the snapshot branches using the natural byte order of their name

 2. for each branch, with a given *name*, add a sequence of bytes composed of
    - the type of the branch target:
     	 - `"content"`, `"directory"`, `"revision"`, `"release"` or `"snapshot"` for each corresponding object type
     	 - `"alias"` for branches referencing another branch;
     	 - `"dangling"` for dangling branches (TODO: is this needed for this spec?)
    - an ASCII space
    - the branch name (as raw bytes)
    - a NULL byte
    - the length of the target identifier, as an ascii-encoded decimal number
      (`"20"` for intrinsic identifiers, `"0"` for dangling
      branches, the length of the name of the target branch for branch aliases)
    - an ASCII colon (`":"`)
    - the identifier of the target object pointed at by the branch:
     	- for contents, directories, revisions, releases or snapshots: their intrinsic
     	  identifier as a string of 20 bytes
     	- for branch aliases, the name of the target branch (as a string of bytes)
     	- for dangling branches, the empty string

   Note that, akin to the serialization of directories, there is no separator between
   entries. Because of alias branches, target identifiers are of arbitrary
   length and are length-encoded to avoid ambiguity.

The intrinsic identifier of the snapshot is the SHA1 of the byte sequence obtained by juxtaposing

 - the ASCII string `"snapshot"` (without quotes),
 - an ASCII space,
 - the length of the previously obtained serialization as ASCII-encoded decimal digits,
 - a NULL byte,
 - and the previously obtained serialization.



As an example, `swh:1:snp:c7c108084bc0bf3d81436bf980b46e98bd338453` is the SWHID computed from
[a snapshot of the entire Darktable Git repository](https://archive.softwareheritage.org/c7c108084bc0bf3d81436bf980b46e98bd338453) as it was on 4 May 2017 on GitHub.

## Note on compatibility with Git

SWHIDs for contents, directories, revisions, and releases are, at present,
compatible with the way the current version of [Git](https://git-scm.com/) 
proceeds for 
[computing identifiers](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects) for
its objects. The `<object_id>` part of a SWHID for a content object is the Git
blob identifier of any file with the same content; for a revision it is the Git
commit identifier for the same revision, etc.  This is not the case for snapshot
identifiers, as Git does not have a corresponding object type.

Git compatibility is practical, but incidental and is not guaranteed to be
maintained in future versions of this standard, nor for different versions of
Git.

