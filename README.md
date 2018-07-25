s3-parallel-put  Parallel uploads to Amazon AWS S3
==================================================

s3-parallel-put speeds the uploading of many small keys to Amazon AWS S3 by
executing multiple PUTs in parallel.


Dependencies
------------

* Python 2.X
* [Boto](http://code.google.com/p/boto/)
* [python-magic](https://github.com/ahupp/python-magic)


Installation
------------

```bash
# Debian family
apt-get update && apt-get -y install python-pip
pip install boto
pip install python-magic
wget -o /usr/bin/s3-parallel-put https://raw.githubusercontent.com/mishudark/s3-parallel-put/master/s3-parallel-put
chmod +x /usr/bin/s3-parallel-put
```

Usage
-----

The program reads your credentials from the environment variables
`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.

```bash
s3-parallel-put --bucket=BUCKET --prefix=PREFIX SOURCE
```

Keys are computed by combining `PREFIX` with the path of the file, starting
from `SOURCE`.  Values are file contents.

#### Options:
* `-h, --help` — show help message

##### S3 options:
* `--bucket=BUCKET` — set bucket
* `--bucket_region=BUCKET_REGION` — set bucket region if not in us-east-1 (default new bucket region)
* `--host=HOST` — set AWS host name
* `--secure` and `--insecure` control whether a secure connection is used
  
##### Source options:
* `--walk=MODE` — set walk mode (filesystem or tar)
* `--exclude=PATTERN` — exclude files matching PATTERN
* `--include=PATTERN` — don't exclude files matching PATTERN
* `--rm-src` -- remove the source file from disk
  
##### Put options:
* `--content-type=CONTENT-TYPE` — sets the `Content-Type` header, set to "guess" to guess based on file name or "magic" to guess by filename and libmagic
* `--gzip` — compresses common text files and sets the `Content-Encoding` header to `gzip`
* `--gzip-type=GZIP_TYPE` — if `--gzip` is set, sets what content-type to gzip, defaults to a list of known text content types, "all" will gzip everything. Specify multiple times for multiple content types (eg.`--gzip-type=guess --gzip-type="image/svg+xml"`) [default: "guess"]
* `--put=MODE` — sets the heuristic used for deciding whether to upload a file or not.  Valid modes are:
  * `add` set the key's content if the key is not already present.
  * `stupid` always set the key's content.
  * `update` set the key's content if the key is not already present and its
  content has changed (as determined by its ETag).
  * `copy`<br>
  The default heuristic is `update`.  If you know that the keys are not already present then `stupid` is fastest (it avoids an extra HEAD request for each key).  If you know that some keys are already present and that they have the correct values, then `add` is faster than `update` (it avoids calculating the MD5 sum of the content on the client side).
  
  
* `--prefix=PREFIX` — set key prefix
* `--resume=FILENAME` — resume from log file
* `--grant=GRANT` — A [Canned ACL](http://docs.amazonwebservices.com/AmazonS3/latest/dev/ACLOverview.html#CannedACL) policy to be applied to each file uploaded. Choices: `private`, `public-read`, `public-read-write`, `authenticated-read`, `bucket-owner-read`, `bucket-owner-full-control`, `log-delivery-write`
* `--header=HEADER:VALUE` — adds an arbitrary header to the S3 file. This option can be specified multiple times.
* `--encrypt-key` — use server side encryption

##### Logging options:
* `--log-filename=FILENAME` — set log filename
* `-q, --quiet` — less output
* `-v, --verbose` — more output to be printed, including progress of individual files

##### Debug and performance tuning options:
* `--dry-run` — don't write to S3. Causes the program to print what it would do, but not to upload any files. It is strongly recommended that you test the program with this option before transferring any real data.
* `--limit=N` — set maximum number of keys to put. Causes the program to upload no more than N files.  Combined
with `--dry-run`, this is also useful for testing.
* `--processes=N` —  sets the number of parallel upload processes


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
* Automatically parallelize uploads of large files by splitting into chunks.


Related projects
----------------

* [JetS3t](http://www.jets3t.org/)
* [s3cmd](http://s3tools.org/s3cmd)
* [sync_media_s3](http://code.google.com/p/django-command-extensions/wiki/sync_media_s3)


Licence
-------

The MIT License (MIT)

Copyright (c) 2011-2014 Tom Payne

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

vim: set spell spelllang=en textwidth=76:
