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

*TODO* For each one: Description, Metadata, Intent, Examples

## 5.1 Contents

A *content* is an uninterpreted byte sequence, typically, the content of a file.
For this type of object the intrinsic identifier is the `sha1_git` hash of it,
i.e. the SHA1 of the byte sequence obtained by juxtaposing
 - the ASCII string `"blob"` (without quotes),
 - a space,
 - the length of the content as decimal digits,
 - a NULL byte,
 - and the actual content of the file.

No metadata is used for this type of object (in particular, notice that there
is no file *name* mentioned here).

As an example, `swh:1:cnt:94a9ed024d3859793618152ea559a168bbcbb5e2` is the SWHID computed from [the full text of the GPL3 license](https://archive.softwareheritage.org/swh:1:cnt:94a9ed024d3859793618152ea559a168bbcbb5e2)

## 5.2 Directories ##

Directories are data structures commonly used in hierarchical file systems to group together files and other directories, and to hold relevant metadata about them, in the form of directory entries.

This version of the SWHID standard adopts the same convention as the popular *git* version control system, and only takes into account as metadata the name of the directory entries and a simplified representation of the access rights.

In order to compute the intrinsic identifier of a directory, it is necessary to compute first the SWHID of each object listed in the directory.

*TODO*: make the following more formal

Then one proceeds to create a textual representation of the directory as follows:

 1. for each *content* entry, with a given *name*, add a single line composed of
     + the access rights, encoded as a sequence of six decimal digits,
   	 + a space,
   	 + the ASCII string `"blob"` (without quotes),
   	 + a space,
   	 + the *intrinsic identifier* of the *content*
   	 + a space,
   	 + the *name* (encoded, how?) as a string (*TODO*: clarify)
   	 + a line terminator

 2. for each *directory* entry, with a given *name*, add a single line composed of
   	 + the access rights, encoded as a sequence of six decimal digits,
   	 + a space,
   	 + the ASCII string `"tree"` (without quotes),
   	 + a space,
   	 + the *intrinsic identifier* of the *directory*
   	 + a space,
   	 + the *name* (encoded, how?) as a string (*TODO*: clarify)
   	 + a line terminator

 3. sort the lines using the following algorithm
     + *TODO* describe the sorting algorithm with all its quirks

The intrinsic identifier of the directory is the `sha1_git` of the
resulting textual representation.

As an example, `swh:1:dir:d198bc9d7a6bcf6db04f476d29314f157507d505` is the
identifier
of
[a directory containing the source code of the darktable photography application](https://archive.softwareheritage.org/swh:1:dir:d198bc9d7a6bcf6db04f476d29314f157507d505) as
a given point in time of its development on May 4th 2017.

## 5.3 Revisions

*TODO*

As an example, `swh:1:rev:309cf2674ee7a0749978cf8265ab91a60aea0f7d` points to 
[a commit in the development history of Darktable](https://archive.softwareheritage.org/swh:1:rev:309cf2674ee7a0749978cf8265ab91a60aea0f7d), dated 16 January 2017, that added undo/redo supports for masks.


## 5.4 Releases

*TODO*

As an example, `swh:1:rel:22ece559cc7cc2364edc5e5593d63ae8bd229f9f` points to
the [Darktable release 2.3.0](https://archive.softwareheritage.org/swh:1:rel:22ece559cc7cc2364edc5e5593d63ae8bd229f9f), dated 24 December 2016.


## 5.5 Snapshots

Any kind of software origin offers multiple pointers to the “current” state of a development project. In the case of VCS this is reflected by branches (e.g., master, development, but also so called feature branches dedicated to extending the software in a specific direction); in the case of package distributions by notions such as suites that correspond to different maturity levels of individual packages (e.g., stable, development, etc.).

A “snapshot” of a given software origin records all entry points found there and where each of them was pointing at the time. For example, a snapshot object might track the commit where the master branch was pointing to at any given time, as well as the most recent release of a given package in the stable suite of a FOSS distribution.

Practically, a snapshot is a list of named branches pointing at objects of any of the known types (content, directory, revision, release or snapshot). A branch can also be an alias to another (named) branch, (FIXME?) for instance the default `"HEAD"` branch can point at another, more specific, `"refs/heads/main"` branch.

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

The intrinsic identifier of the snapshot is the SHA1 of the of the byte sequence obtained by juxtaposing
 - the ASCII string `"snapshot"` (without quotes),
 - an ASCII space,
 - the length of the previously obtained serialization as ASCII-encoded decimal digits,
 - a NULL byte,
 - and the previously obtained serialization.



As an example, `swh:1:snp:c7c108084bc0bf3d81436bf980b46e98bd338453` points to a
[snapshot of the entire Darktable Git repository](https://archive.softwareheritage.org/c7c108084bc0bf3d81436bf980b46e98bd338453) as it was on 4 May 2017 on GitHub.

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

