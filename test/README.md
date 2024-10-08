c-ares Unit Test Suite
======================

This directory holds unit tests for the c-ares library.  To build the tests:

 - Build the main c-ares library first, in the directory above this.  To
   enable tests of internal functions, configure the library build to expose
   hidden symbols with `./configure --disable-symbol-hiding`.
 - Generate a `configure` file by running `autoreconf -iv` (which requires
   a local installation of
   [autotools](https://www.gnu.org/software/automake/manual/html_node/Autotools-Introduction.html)).
 - `./configure`
 - `make`
 - Run the tests with `./arestest`, or `./arestest -v` for extra debug info.

Points to note:

 - The tests are written in C++11, and so need a C++ compiler that supports
   this.  To avoid adding this as a requirement for the library, the
   configuration and build of the tests is independent from the library.
 - The tests include some live queries, which will fail when run on a machine
   without internet connectivity.  To skip live tests, run with
   `./arestest --gtest_filter=-*.Live*`.
 - The tests include queries of a mock DNS server.  This server listens on port
   5300 by default, but the port can be changed with the `-p 5300` option to
   `arestest`.


Test Types
----------

The test suite includes various different types of test.

 - There are live tests (`ares-test-live.cc`), which assume that the
   current machine has a valid DNS setup and connection to the
   internet; these tests issue queries for real domains but don't
   particularly check what gets returned.  The tests will fail on
   an offline machine.
 - There are some mock tests (`ares-test-mock.cc`) that set up a fake DNS
   server and inject its port into the c-ares library configuration.
   These tests allow specific response messages to be crafted and
   injected, and so are likely to be used for many more tests in
   future.
    - To make this generation/injection easier, the `dns-proto.h`
      file includes C++ helper classes for building DNS packets.
 - Other library entrypoints that don't require network activity
   (e.g. `ares_parse_*_reply`) are tested directly.
 - A couple of the tests use a helper method of the test fixture to
   inject memory allocation failures, using a recent change to the
   c-ares library that allows override of `malloc`/`free`.
 - There are some tests of the internal entrypoints of the library
   (`ares-test-internal.c`), but these are only enabled if the library
   was configured with `--disable-symbol-hiding` and/or
   `--enable-expose-statics`.
 - There is also an entrypoint to allow Clang's
   [libfuzzer](http://llvm.org/docs/LibFuzzer.html) to drive
   the packet parsing code in `ares_parse_*_reply`, together with a
   standalone wrapper for it (`./aresfuzz`) to allow use of command
   line fuzzers (such as [afl-fuzz](http://lcamtuf.coredump.cx/afl/))
   for further [fuzz testing](#fuzzing).


Code Coverage Information
-------------------------

To generate code coverage information:

 - Configure both the library and the tests with `./configure
   --enable-code-coverage` before building. This requires the relevant code
   coverage tools ([gcov](https://gcc.gnu.org/onlinedocs/gcc/Gcov.html),
   [lcov](http://ltp.sourceforge.net/coverage/lcov.php)) to be installed locally.
 - Run the tests with `test/arestest`.
 - Generate code coverage output with `make code-coverage-capture` in the
   library directory (i.e. not in `test/`).

