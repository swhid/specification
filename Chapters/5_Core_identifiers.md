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

The intrinsic identifier of the directory is the SHA1 of the of the byte sequence obtained by juxtaposing
 - the ASCII string `"tree"` (without quotes),
 - an ASCII space,
 - the length of the previously obtained serialization as ASCII-encoded decimal digits,
 - a NULL byte,
 - and the previously obtained serialization.

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

*TODO*

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

