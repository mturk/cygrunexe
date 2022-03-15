# cygwrun

Run windows applications under Posix environment

## Overview

Cygwin uses posix paths and environments which makes most of
the standard windows programs to fail because of path mismatch.
The traditional way of handling that is using Cygwin cygpath
utility which translates Cygwin (posix) paths to their windows
equivalents from shell.

For example a standard usage would be:
```
program.exe "--f1=`cygpath -w /tmp/f1`" "`cygpath -w /usr/f1`" ...
```
This can become very complex and it requires that the shell
script is aware it runs inside the Cygwin environment.

cygwrun utility does that automatically by replacing each posix
argument that contains path element with its windows equivalent.
It also replaces paths in the environment variable values making
sure the multiple path elements are correctly separated using
windows path separator `;`.

Using cygwrun the upper example would become:
```
cygwrun program.exe --f1=/tmp/f1 /usr/f1 ...
```
Before starting `program.exe` cygwrun converts all command line
and environment variables to windows format.

## Usage

Here is what the usage screen displays
```
Usage cygwrun [OPTIONS]... PROGRAM [ARGUMENTS]...
Execute PROGRAM [ARGUMENTS]...

Options are:
 -r <DIR>  use DIR as posix root
 -w <DIR>  change working directory to DIR before calling PROGRAM
 -k        keep extra posix environment variables.
 -s        do not translate environment variables.
 -q        do not print errors to stderr.
 -v        print version information and exit.
 -h        print this screen and exit.
 -p        print arguments instead executing PROGRAM.
 -e        print current environment block end exit.
           if defined, only print variables that begin with ARGUMENTS.

```

Note that `-p` or `-e` option terminate options processing,
so make sure to define any other options before them.

## Posix root

Posix root is used to replace posix parts with posix environment root
location inside Windows environment.

Use `-r <directory>` command line option to setup the install location
of the current posix subsystem.

In case the `-r <directory>` was not specified, the program will
check the following environment variables;

First it will check private environment variable `_CYGWRUN_POSIX_ROOT`.
Then it will check `CYGWIN_ROOT` and then `POSIX_ROOT` variables.
If none of them were defined, the `C:\cygwin64` or `C:\cygwin`
will be used, if there is `C:\cygwin64\etc\fstab` or `C:\cygwin\etc\fstab` file present.

Make sure that you provide a correct posix root since it will
be used as prefix to `/usr, /bin, /tmp` constructing an actual
Windows path.


For example, if Cygwin is installed inside `C:\cygwin64` you
can set either environment variable

```sh
    $ export CYGWIN_ROOT=C:/cygwin64
    ...
    $ cygwrun ... -f1=/usr/local
```

Or declare it on command line

```sh
    $ cygwrun -r C:/cygwin64 ... -f1=/usr/local
```

In both cases `-f1 parameter` will evaluate to `-f1=C:\cygwin64\usr\local`

## Environment variables

Since Cygwrun presumes that it's called by some Cygwin process,
it translates current environment variables from posix to windows
paths automatically.

Cygwin program that calls Cygwrun, will already translate
most of the known environment variables to Windows format,
but not others. Cygwrun will translate all environment
variables which value is a valid posix path to Windows format.

Note that some environment variables are allways removed from the
current environment variable list that is passed to chiled process,
like `OLDPWD` or `PS1`.

The full list of variables that are allways removed is defined
with `removeenv[]` array in [cygwrun.c](cygwrun.c) source file.

For example, if environment variable contains valid posix path(s)
it will be translated to windows path(s).

```sh
    $ export FOO=/usr:/sbin:../dir
    $ cygwrun -e FOO
    $ FOO=C:\cygwin64\usr;C:\cygwin64\sbin;..\dir
```

However in case the environment variable value contains path element that
cannot be translated to windows path, the original value is preserved,
regardless if all other path elements can be translated.

```sh
    $ export FOO=/usr:/sbin:/unknown:../dir
    $ cygwrun -e FOO
    $ FOO=/usr:/sbin:/unknown:../dir
```



## License

The code in this repository is licensed under the [Apache-2.0 License](LICENSE.txt).
