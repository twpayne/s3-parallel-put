s3-parallel-put  Parallel uploads to Amazon AWS S3
==================================================

s3-parallel-put speeds the uploading of many small keys to Amazon AWS S3 by
executing multiple PUTs in parallel.


Dependencies
------------

* [Boto](http://code.google.com/p/boto/)


Usage
-----

The program reads your credentials from the environment variables
`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.

```bash
s3-parallel-put --bucket=BUCKET --prefix=PREFIX SOURCE
```

Keys are computed by combining `PREFIX` with the path of the file, starting
from `SOURCE`.  Values are file contents.

There are a few other options:

`--dry-run` causes the program to print what it would do, but not to upload
any files.  It is strongly recommended that you test the program with this
option before transferring any real data.

`--limit=N` causes the program to upload no more than N files.  Combined
with `--dry-run`, this is also useful for testing.

`--put=MODE` sets the heuristic used for deciding whether to upload a file
or not.  Valid modes are:

* `add` set the key's content if the key is not already present.

* `stupid` always set the key's content.

* `update` set the key's content if the key is not already present and it's
  content has changed (as determined by its ETag).

The default heuristic is `update`.  If you know that the keys are not
already present then `stupid` is fastest (it avoids an extra HEAD request
for each key).  If you know that some keys are already present and that they
have the correct values, then `add` is faster than `update` (it avoids
calculating the MD5 sum of the content on the client side).

`--content-type=CONTENT-TYPE` sets the `Content-Type` header.

`--gzip` compresses all values and sets the `Content-Encoding` header to
`gzip`.

`--processes=N` sets the number of parallel upload processes.

`--verbose` causes more output to be printed, including progress of individual files.

`--quiet` causes less output.

`--secure` and `--insecure` control whether a secure connection is used.

`--grant` applies a
[Canned ACL](http://docs.amazonwebservices.com/AmazonS3/latest/dev/ACLOverview.html#CannedACL)
to all files uploaded.


Architecture
------------

* A walker process generates (filename, key_name) pairs and inserts them in
  `put_queue`.
* Multiple putter processes consume these pairs in parallel, uploading the
  files to S3 and sending file-by-file statistics to `stat_queue`.
* A statter process consumes these file-by-file statistics and generates
  summary statistics.


Bugs
----

* Limited error checking.


To Do
-----

* Update documentation.

* Automatically parallelize uploads of large files by splitting into chunks.


Related projects
----------------

* [JetS3t](http://www.jets3t.org/)
* [s3cmd](http://s3tools.org/s3cmd)
* [sync_media_s3](http://code.google.com/p/django-command-extensions/wiki/sync_media_s3)


Licence
-------

Copyright (C) 2011  Tom Payne

This program is free software: you can redistribute it and/or modify it
under the terms of the GNU General Public License as published by the Free
Software Foundation, either version 3 of the License, or (at your option) any
later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY
WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.  See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with
this program.  If not, see <http://www.gnu.org/licenses/>.


vim: set spell spelllang=en textwidth=76:
