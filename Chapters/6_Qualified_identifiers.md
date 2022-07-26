# 6 Qualified identifiers

### Qualifiers

A qualified, or full, SWHID is composed of a core SWHID identifier, and a
sequence of qualifiers that may identify subparts of a software artefact
(*fragment qualifiers*), or provide additional context on the software artefact
(*context qualifiers*). The `;` character is used a separator between the core
identifier and the optional qualifiers, as well as between qualifiers. Each
qualifier is specified as a key/value pair, using `=` as a separator.

The following *context qualifiers* are available:


*TODO* For each one: Description, Metadata, Intent, Examples

## 6.1 Fragment qualifiers

### 6.1.1 Lines qualifier
This qualifier allows to designate a line range inside a content.
The range can be a single line number, or a pair of line numbers
separated by `-`.

For example, [`swh:1:cnt:4d99d2d18326621ccdd70f5ea66c2e2ac236ad8b;lines=9-15`](https://archive.softwareheritage.org/swh:1:cnt:4d99d2d18326621ccdd70f5ea66c2e2ac236ad8b;lines=9-15)
designates the function `generate_intput_stream` that is found at lines 9 to 15 of the *content* with core SWHID `swh:1:cnt:4d99d2d18326621ccdd70f5ea66c2e2ac236ad8b`.

Notice that the notion of "line number" is not always well defined: the content
may be a binary file, or a file that uses non standard line termination
character(s).

*TODO*: decide whether we prescribe to use only the Unix line terminator (this will add extra ^M to DOS/Windows lines)


## 6.2 Context qualifiers

### 6.2.1 Origin qualifier
This qualifier allows to declare the *software origin* where the
object has been found or observed, as an URI.

For example, [`swh:1:cnt:4d99d2d18326621ccdd70f5ea66c2e2ac236ad8b;origin=https://gitorious.org/ocamlp3l/ocamlp3l_cvs.git`](https://archive.softwareheritage.org/swh:1:cnt:4d99d2d18326621ccdd70f5ea66c2e2ac236ad8b;origin=https://gitorious.org/ocamlp3l/ocamlp3l_cvs.git)
indicates that the content seen previously with the function `generate_intput_stream` has
been seen in the Git repository at `https://gitorious.org/ocamlp3l/ocamlp3l_cvs.git`

This qualifier may be helpful to get hold of the full repository where a
content has been found, but there is no guarantee of success, as an origin
can change or disappear over time (it is the case of the example above, as
gitorious.org was shut down in 2015).

### 6.2.2 Visit qualifier
This qualifier allows to add the core SWHID identifier of the *snapshot* 
of the repository where the object has been found or observed.

For example, [`swh:1:cnt:4d99d2d18326621ccdd70f5ea66c2e2ac236ad8b;origin=https://gitorious.org/ocamlp3l/ocamlp3l_cvs.git;visit=swh:1:snp:d7f1b9eb7ccb596c2622c4780febaa02549830f9`](https://archive.softwareheritage.org/swh:1:cnt:4d99d2d18326621ccdd70f5ea66c2e2ac236ad8b;origin=https://gitorious.org/ocamlp3l/ocamlp3l_cvs.git;visit=swh:1:snp:d7f1b9eb7ccb596c2622c4780febaa02549830f9)
indicates that the content seen previously with the function `generate_intput_stream` has
been seen in the Git repository at `https://gitorious.org/ocamlp3l/ocamlp3l_cvs.git`, when
its full state had the SWHID core identifier `swh:1:snp:d7f1b9eb7ccb596c2622c4780febaa02549830f9`. 

*TODO*: explain where `visit` comes from?

### 6.2.3 Anchor qualifier
This qualifier is used in conjunction with the `path` qualifier.
It allows to identify a node in the Merkle DAG relative to which
a *path to the object* is specified, as the core identifier of a directory,
a revision, a release or a snapshot. See below for an example.

### 6.2.4 Path qualifier
This qualifier allows to declare the *absolute file path*, from the *root
directory* associated to the *anchor node*, to the object designated by the
core SWHID identifier; when the anchor denotes a directory or a revision,
and almost always when it\'s a release, the root directory is uniquely
determined; when the anchor denotes a snapshot, the root directory is the
one pointed to by `HEAD` branch (possibly indirectly), and undefined if such a
reference is missing.
	
*TODO*: clarify this section (HEAD, branch, etc.)

For example, [`swh:1:cnt:4d99d2d18326621ccdd70f5ea66c2e2ac236ad8b;origin=https://gitorious.org/ocamlp3l/ocamlp3l_cvs.git;visit=swh:1:snp:d7f1b9eb7ccb596c2622c4780febaa02549830f9;anchor=swh:1:rev:2db189928c94d62a3b4757b3eec68f0a4d4113f0;path=/Examples/SimpleFarm/simplefarm.ml`](https://archive.softwareheritage.org/swh:1:cnt:4d99d2d18326621ccdd70f5ea66c2e2ac236ad8b;origin=https://gitorious.org/ocamlp3l/ocamlp3l_cvs.git;visit=swh:1:snp:d7f1b9eb7ccb596c2622c4780febaa02549830f9;anchor=swh:1:rev:2db189928c94d62a3b4757b3eec68f0a4d4113f0;path=/Examples/SimpleFarm/simplefarm.ml)
indicates that the content seen previously with the function `generate_intput_stream` has
been seen in the Git repository at `https://gitorious.org/ocamlp3l/ocamlp3l_cvs.git`, when
its full state had the SWHID core identifier `swh:1:snp:d7f1b9eb7ccb596c2622c4780febaa02549830f9`, and that it is named `simplefarm.ml` in the directory `Simplefarm` contained in the directory `Examples` contained in the root directory associated to the revision with core SWHID `swh:1:rev:2db189928c94d62a3b4757b3eec68f0a4d4113f0`.


### 6.3 Recommendations

We recommend to equip identifiers meant to be shared with as many
qualifiers as possible. While qualifiers may be listed in any order, it
is good practice to present them in the following order:
`origin`, `visit`, `anchor`, `path` or `lines`. Redundant information
should be omitted: for example, if the *visit* is present, and the
*path* is relative to the snapshot indicated there, then the *anchor*
qualifier is superfluous; similarly, if the *path* is empty, it may be
omitted.

Here is an example: [`swh:1:cnt:4d99d2d18326621ccdd70f5ea66c2e2ac236ad8b;origin=https://gitorious.org/ocamlp3l/ocamlp3l_cvs.git;visit=swh:1:snp:d7f1b9eb7ccb596c2622c4780febaa02549830f9;anchor=swh:1:rev:2db189928c94d62a3b4757b3eec68f0a4d4113f0;path=/Examples/SimpleFarm/simplefarm.ml;lines=9-15`](https://archive.softwareheritage.org/swh:1:cnt:4d99d2d18326621ccdd70f5ea66c2e2ac236ad8b;origin=https://gitorious.org/ocamlp3l/ocamlp3l_cvs.git;visit=swh:1:snp:d7f1b9eb7ccb596c2622c4780febaa02549830f9;anchor=swh:1:rev:2db189928c94d62a3b4757b3eec68f0a4d4113f0;path=/Examples/SimpleFarm/simplefarm.ml;lines=9-15)


