CMake-based MinGW-w64 Cross Toolchain
=====================================

This thing’s primary use is to build Windows binaries of mpv.

Prerequisites
-------------

In addition to CMake, you need the usual development stuff (Git, Subversion,
GCC, Binutils, ragel, headers for GMP, MPFR and MPC).

.. note::
    You should also install Ninja and use CMake’s Ninja build file generator.
    It’s not only much faster than GNU Make, but also far less error-prone,
    which is important for this project because CMake’s ExternalProject module
    tends to generate makefiles which confuse GNU Make’s jobserver thingy.

.. note::
    As a build environment, any modern Linux distribution *should* work,
    however I am only testing this on openSUSE, which (as of November 2014)
    is using a 4.8 series GCC by default. Feel free to contribute fixes for
    other environments.

    If you are looking for VM images with everything set up to work with this:
    `<https://github.com/lachs0r/mingw-w64-env>`_


Building Software
-----------------

To set up the build environment, create a directory to store build files in::

    mkdir build-64
    cd build-64

Once you’ve changed into that directory, run CMake, e.g.::

    cmake -DTARGET_ARCH=x86_64-w64-mingw32 -DCMAKE_INSTALL_PREFIX=prefix -G Ninja ..

Once that’s done, you’re ready to build stuff. For example, to build mpv and
all of its dependencies::

    ninja mpv

This will take a while (about 20 minutes on my machine).

.. note::
    The mpv package has some additional steps to generate files ready
    for distribution instead of installing it to the prefix.

.. note::
    If you wish to disable automatic updates of packages pulled from
    development sources, use ``-DENABLE_VCS_UPDATES=false`` on the CMake
    command line.


Unpackaged Stuff
~~~~~~~~~~~~~~~~

Using the toolchain to build stuff which doesn’t have a package is usually
very easy. There are two generated files in your build directory to help with
this: “exec” and “toolchain.cmake”.

For most software (i.e. almost everything that uses GNU Autotools), you can
use “exec” with the configure command:

    ~/mingw/build-64/exec ./configure --prefix=~/mingw/prefix-64/mingw --host=x86_64-w64-mingw32

An alternative is to run “source ~/mingw/build-64/exec” to set all the required
environment variables in your current session.

For software that uses CMake, you can use “toolchain.cmake” like this:

    cmake -DCMAKE_TOOLCHAIN_FILE=~/mingw/build-64/toolchain.cmake -DCMAKE_INSTALL_PREFIX=~/mingw/prefix-64/mingw

In general, it is advisable to use static linking when building for Windows.
To do that, use --disable-shared and/or --enable-static with Autotools-based
configure scripts.

CMake doesn’t have a standard way to achieve this, so you’re on your own.

.. note::
    It’s usually easy to make CMake projects link statically if they don’t have
    an option for it already. If you need an example, look at the patches for
    ``game-music-emu``.


Creating Packages
~~~~~~~~~~~~~~~~~

To add a new package, create a new ``.cmake`` file in the ``packages``
directory (just look at how the existing packages work) and add it to the
list in ``packages/CMakeLists.txt`` (they must appear after their
dependencies).

See the CMake documentation on the ExternalProject module for further info.
