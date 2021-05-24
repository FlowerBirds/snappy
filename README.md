# Snappy
## A fast compressor/decompressor library.

[![Build Status](https://travis-ci.org/google/snappy.svg?branch=master)](https://travis-ci.org/google/snappy)
[![Build status](https://ci.appveyor.com/api/projects/status/t9nubcqkwo8rw8yn/branch/master?svg=true)](https://ci.appveyor.com/project/pwnall/leveldb)

Introduction
============

Snappy is a compression/decompression library. It does not aim for maximum
compression, or compatibility with any other compression library; instead,
it aims for very high speeds and reasonable compression. For instance,
compared to the fastest mode of zlib, Snappy is an order of magnitude faster
for most inputs, but the resulting compressed files are anywhere from 20% to
100% bigger. (For more information, see "Performance", below.)

Snappy has the following properties:

 * Fast: Compression speeds at 250 MB/sec and beyond, with no assembler code.
   See "Performance" below.
 * Stable: Over the last few years, Snappy has compressed and decompressed
   petabytes of data in Google's production environment. The Snappy bitstream
   format is stable and will not change between versions.
 * Robust: The Snappy decompressor is designed not to crash in the face of
   corrupted or malicious input.
 * Free and open source software: Snappy is licensed under a BSD-type license.
   For more information, see the included COPYING file.

Snappy has previously been called "Zippy" in some Google presentations
and the like.


Performance
===========

Snappy is intended to be fast. On a single core of a Core i7 processor
in 64-bit mode, it compresses at about 250 MB/sec or more and decompresses at
about 500 MB/sec or more. (These numbers are for the slowest inputs in our
benchmark suite; others are much faster.) In our tests, Snappy usually
is faster than algorithms in the same class (e.g. LZO, LZF, QuickLZ,
etc.) while achieving comparable compression ratios.

Typical compression ratios (based on the benchmark suite) are about 1.5-1.7x
for plain text, about 2-4x for HTML, and of course 1.0x for JPEGs, PNGs and
other already-compressed data. Similar numbers for zlib in its fastest mode
are 2.6-2.8x, 3-7x and 1.0x, respectively. More sophisticated algorithms are
capable of achieving yet higher compression rates, although usually at the
expense of speed. Of course, compression ratio will vary significantly with
the input.

Although Snappy should be fairly portable, it is primarily optimized
for 64-bit x86-compatible processors, and may run slower in other environments.
In particular:

 - Snappy uses 64-bit operations in several places to process more data at
   once than would otherwise be possible.
 - Snappy assumes unaligned 32 and 64-bit loads and stores are cheap.
   On some platforms, these must be emulated with single-byte loads
   and stores, which is much slower.
 - Snappy assumes little-endian throughout, and needs to byte-swap data in
   several places if running on a big-endian platform.

Experience has shown that even heavily tuned code can be improved.
Performance optimizations, whether for 64-bit x86 or other platforms,
are of course most welcome; see "Contact", below.


Building
========

You need the CMake version specified in [CMakeLists.txt](./CMakeLists.txt)
or later to build:

```bash
git submodule update --init
mkdir build
cd build && cmake ../ && make
```

If you want to build shared library **libsnappy.so**, you should add **BUILD_SHARED_LIBS** option, like
```bash
[root@localhost build]# cmake -DBUILD_SHARED_LIBS=ON ../
...
...

[root@localhost build]# make
[  2%] Building CXX object CMakeFiles/snappy.dir/snappy-c.cc.o
[  4%] Building CXX object CMakeFiles/snappy.dir/snappy-sinksource.cc.o
[  6%] Building CXX object CMakeFiles/snappy.dir/snappy-stubs-internal.cc.o
[  9%] Building CXX object CMakeFiles/snappy.dir/snappy.cc.o

[ 11%] Linking CXX shared library libsnappy.so
[ 11%] Built target snappy
[ 13%] Building CXX object CMakeFiles/snappy_test_support.dir/snappy-test.cc.o
[ 16%] Building CXX object CMakeFiles/snappy_test_support.dir/snappy_test_data.cc.o
[ 18%] Linking CXX shared library libsnappy_test_support.so
[ 18%] Built target snappy_test_support


[root@localhost build]# ll
总用量 644
drwxr-xr-x  2 root root   4096 5月  24 15:05 bin
drwxr-xr-x  2 root root   4096 5月  24 15:10 cmake
-rw-r--r--  1 root root  25622 5月  24 15:05 CMakeCache.txt
drwxr-xr-x 10 root root   4096 5月  24 15:10 CMakeFiles
-rw-r--r--  1 root root   5534 5月  24 15:05 cmake_install.cmake
-rw-r--r--  1 root root   1825 5月  24 15:05 config.h
-rw-r--r--  1 root root    573 5月  24 15:05 CTestTestfile.cmake
drwxr-xr-x  2 root root   4096 5月  24 15:06 lib
lrwxrwxrwx  1 root root     14 5月  24 15:05 libsnappy.so -> libsnappy.so.1
lrwxrwxrwx  1 root root     18 5月  24 15:05 libsnappy.so.1 -> libsnappy.so.1.1.9
-rwxr-xr-x  1 root root 149888 5月  24 15:05 libsnappy.so.1.1.9
-rwxr-xr-x  1 root root  35344 5月  24 15:05 libsnappy_test_support.so
-rw-r--r--  1 root root  19848 5月  24 15:10 Makefile
-rwxr-xr-x  1 root root 122520 5月  24 15:06 snappy_benchmark
-rw-r--r--  1 root root   2579 5月  24 15:05 snappy-stubs-public.h
-rwxr-xr-x  1 root root  60728 5月  24 15:05 snappy_test_tool
-rwxr-xr-x  1 root root 193408 5月  24 15:05 snappy_unittest
drwxr-xr-x  4 root root   4096 5月  24 15:05 third_party


```

Usage
=====

Note that Snappy, both the implementation and the main interface,
is written in C++. However, several third-party bindings to other languages
are available; see the [home page](docs/README.md) for more information.
Also, if you want to use Snappy from C code, you can use the included C
bindings in snappy-c.h.

To use Snappy from your own C++ program, include the file "snappy.h" from
your calling file, and link against the compiled library.

There are many ways to call Snappy, but the simplest possible is

```c++
snappy::Compress(input.data(), input.size(), &output);
```

and similarly

```c++
snappy::Uncompress(input.data(), input.size(), &output);
```

where "input" and "output" are both instances of std::string.

There are other interfaces that are more flexible in various ways, including
support for custom (non-array) input sources. See the header file for more
information.


Tests and benchmarks
====================

When you compile Snappy, the following binaries are compiled in addition to the
library itself. You do not need them to use the compressor from your own
library, but they are useful for Snappy development.

* `snappy_benchmark` contains microbenchmarks used to tune compression and
  decompression performance.
* `snappy_unittests` contains unit tests, verifying correctness on your machine
  in various scenarios.
* `snappy_test_tool` can benchmark Snappy against a few other compression
  libraries (zlib, LZO, LZF, and QuickLZ), if they were detected at configure
  time. To benchmark using a given file, give the compression algorithm you want
  to test Snappy against (e.g. --zlib) and then a list of one or more file names
  on the command line.

If you want to change or optimize Snappy, please run the tests and benchmarks to
verify you have not broken anything.

The testdata/ directory contains the files used by the microbenchmarks, which
should provide a reasonably balanced starting point for benchmarking. (Note that
baddata[1-3].snappy are not intended as benchmarks; they are used to verify
correctness in the presence of corrupted data in the unit test.)


Contact
=======

Snappy is distributed through GitHub. For the latest version and other
information, see https://github.com/google/snappy.
