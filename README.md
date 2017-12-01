## libsodium's `Findsodium.cmake` ignores `sodium_USE_STATIC_LIBS=ON`

## Platform / Environment
This has been tested on Debian GNU Linux only.

## Description
[libsodium](https://github.com/jedisct1/libsodium) provides
[a cmake module](https://github.com/jedisct1/libsodium/blob/master/contrib/Findsodium.cmake).
A [recent PR](https://github.com/jedisct1/libsodium/pull/621) has been merged to improve static
linking but it doesn't work in the present simple elementary use-case.

## Steps to reproduce

```sh
git clone https://github.com/nibua-r/sodium-issue-621.git
cd sodium-issue-621
cmake -E make_directory build
cd build
cmake -Dsodium_USE_STATIC_LIBS=ON ..
make
ldd ./dummy
```

## Expected and actual result

Expected: the project uses `libsodium.a` and `ldd` shows no reference to `libsodium.so` since
libsodium is statically linked with our `dummy` executable.

Actual result:
```sh
cmake  -Dsodium_USE_STATIC_LIBS=ON ..
-- The C compiler identification is GNU 7.2.1
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Found sodium: /usr/lib/x86_64-linux-gnu/libsodium.so
CMake Warning (dev) in CMakeLists.txt:
  No cmake_minimum_required command is present.  A line of code such as

    cmake_minimum_required(VERSION 3.9)

  should be added at the top of the file.  The version specified may be lower
  if you wish to support older CMake versions for this project.  For more
  information run "cmake --help-policy CMP0000".
This warning is for project developers.  Use -Wno-dev to suppress it.

-- Configuring done
-- Generating done
-- Build files have been written to: /home/renaud/clones/github/build/sodium-issue-621/build

ldd ./dummy
    linux-vdso.so.1 (0x00007ffd0c8f6000)
    libsodium.so.18 => /usr/lib/x86_64-linux-gnu/libsodium.so.18 (0x00007f80b2dd6000)
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f80b2a39000)
    libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f80b281c000)
    /lib64/ld-linux-x86-64.so.2 (0x00007f80b3249000)
```

`dummy` is linked against `libsodium.so`.

## Workaround / Possible solution(?)

`sodium_PKG_STATIC_LIBRARIES` is set to `"sodium"` and the `NOT DEFINED sodium_PKG_STATIC_LIBRARIES`
condition is never triggered. I can't tell if `pkg-config` or `cmake` are doing wrong but a simple
workaround is to always set `sodium_PKG_STATIC_LIBRARIES` to `libsodium.a` if
`sodium_USE_STATIC_LIBS` is true/on.

```diff
diff --git a/cmake/Findsodium.cmake b/cmake/Findsodium.cmake
index ec01ac4..4cd9bf4 100644
--- a/cmake/Findsodium.cmake
+++ b/cmake/Findsodium.cmake
@@ -57,9 +57,7 @@ if (UNIX)
     if(sodium_USE_STATIC_LIBS)
         # if pkgconfig for libsodium doesn't provide
         # static lib info, then override PKG_STATIC here..
-        if (NOT DEFINED sodium_PKG_STATIC_LIBRARIES)
-            set(sodium_PKG_STATIC_LIBRARIES libsodium.a)
-        endif()
+        set(sodium_PKG_STATIC_LIBRARIES libsodium.a)
         set(XPREFIX sodium_PKG_STATIC)
     else()
         set(XPREFIX sodium_PKG)
```
