Hash computation for SWHID version 1
====================================

Below you can find (adapted) content from the relevant doc strings of the
[swh.model.git\_objects Python
module](https://docs.softwareheritage.org/devel/apidoc/swh.model.git_objects.html),
describing how the hash part of SWHID identifiers version 1 are computed.

It is probably not yet self-contained as a specification, we will iterate by
adding missing information as soon as it is identified.


Content objects
---------------

For **contents**, the intrinsic identifier is the SHA1 of a byte sequence
obtained by juxtaposing the ASCII string `"blob"` (without quotes), a space,
the length of the content as decimal digits, a NULL byte, and the actual
content of the file.


Directory objects
-----------------

A directory's identifier is the tree sha1 à la git of a directory listing,
using the following algorithm, which is equivalent to the git algorithm for
trees:

1. Entries of the directory are sorted using the name (or the name with `/`
   appended for directory entries) as key, in bytes order.

2. For each entry of the directory, the following bytes are output:
    - the octal representation of the permissions for the entry (stored in the 'perms' member), which is a representation of the entry type:
	    - b'100644' (int 33188) for files
	    - b'100755' (int 33261) for executable files
	    - b'120000' (int 40960) for symbolic links
	    - b'40000'  (int 16384) for directories
	    - b'160000' (int 57344) for references to revisions
    - an ascii space (b'\x20')
    - the entry's name (as raw bytes), stored in the 'name' member
    - a null byte (b'\x00')
    - the 20 byte long identifier of the object pointed at by the entry, stored in the 'target' member:
	    - for files or executable files: their blob sha1_git
	    - for symbolic links: the blob sha1_git of a file containing the link destination
	    - for directories: their intrinsic identifier
	    - for revisions: their intrinsic identifier

   (Note that there is no separator between entries)


Revision objects
----------------

The fields used for the revision identifier computation are:

- directory
- parents
- author
- author_date
- committer
- committer_date
- extra\_headers or metadata -> extra\_headers
- message

A revision's identifier is the 'git'-checksum of a commit manifest constructed
as follows (newlines are a single ASCII newline character):

	tree <directory identifier>
	[for each parent in parents]
	parent <parent identifier>
	[end for each parents]
	author <author> <author_date>
	committer <committer> <committer_date>
	[for each key, value in extra_headers]
	<key> <encoded value>
	[end for each extra_headers]

	<message>

The directory identifier is the ascii representation of its hexadecimal
encoding.

Author and committer are formatted using the `format_author_data` function (see
dedicated section below).  Dates are formatted with the `format_date` function
(see dedicated section below).

Extra headers are an ordered list of [key, value] pairs. Keys are strings and
get encoded to utf-8 for identifier computation. Values are either byte
strings, unicode strings (that get encoded to utf-8), or integers (that get
encoded to their utf-8 decimal representation).

Multiline extra header values are escaped by indenting the continuation lines
with one ascii space.

If the message is None, the manifest ends with the last header. Else, the
message is appended to the headers after an empty line.

The checksum of the full manifest is computed using the 'commit' git object
type.


Release objects
---------------

**TODO**: to be documented from [`release_git_object`
implementation](https://docs.softwareheritage.org/devel/_modules/swh/model/git_objects.html#release_git_object)


Snapshot objects
----------------

Snapshots are a set of named branches, which are pointers to objects at any
level of the Software Heritage DAG.

As well as pointing to other objects in the Software Heritage DAG, branches can
also be *alias*es, in which case their target is the name of another branch in
the same snapshot, or *dangling*, in which case the target is unknown (and
represented by the `None` value).

A snapshot identifier is a salted sha1 (using the git hashing algorithm with
the `snapshot` object type) of a manifest following the algorithm:

1. Branches are sorted using the name as key, in bytes order.
2. For each branch, the following bytes are output:
   - the type of the branch target:
	   - `content`, `directory`, `revision`, `release` or `snapshot` for the corresponding entries in the DAG;
	   - `alias` for branches referencing another branch;
	   - `dangling` for dangling branches
  - an ascii space (`\\x20`)
  - the branch name (as raw bytes)
  - a null byte (`\\x00`)
  - the length of the target identifier, as an ascii-encoded decimal number (`20` for current intrinsic identifiers, `0` for dangling branches, the length of the target branch name for branch aliases)
  - a colon (`:`)
  - the identifier of the target object pointed at by the branch, stored in the 'target' member:
	  - for contents: their *sha1_git*
	  - for directories, revisions, releases or snapshots: their intrinsic identifier
	  - for branch aliases, the name of the target branch (as raw bytes)
	  - for dangling branches, the empty string

Note that, akin to directory manifests, there is no separator between
entries. Because of symbolic branches, identifiers are of arbitrary length but
are length-encoded to avoid ambiguity.


Auxiliary functions
===================


Author data (`format_author_data`)
-----------

Git authorship data has two components:

- an author specification, usually a name and email, but in practice an
  arbitrary bytestring
- optionally, a timestamp with a UTC offset specification

The authorship data is formatted thus: `name and email` [ `timestamp`
`utc_offset`]

The timestamp is encoded as a (decimal) number of seconds since the UNIX epoch
(1970-01-01 at 00:00 UTC). As an extension to the git format, we support
fractional timestamps, using a dot as the separator for the decimal part.

The utc offset is a number of minutes encoded as '[+-]HHMM'. Note that some
tools can pass a negative offset corresponding to the UTC timezone ('-0000'),
which is valid and is encoded as such.


Timestamps (`format_date`)
----------

Git stores timestamps as an integer number of seconds since the UNIX epoch.

However, Software Heritage stores timestamps as an integer number of
microseconds (postgres type "datetime with timezone").

Therefore, we print timestamps with no microseconds as integers, and timestamps
with microseconds as floating point values. We elide the trailing zeroes from
microsecond values, to "future-proof" our representation if we ever need more
precision in timestamps.
