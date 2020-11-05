title: Fix .so file errors during a CMake or Makefile build
date: 2020-11-05 14:50
modified: 2020-11-05 14:50
category: bug-fix
tags: bug, make, opencv, jetson-nano, cmake, build-errors
slug: fix-so-file-errors
author: satyajit-ghana
summary: When building a library using make, you'll get many kiunds of linking errors and .so file errors, this blog is about few of those kind of errors and how to fix them

# Fix .so file errors during a CMake or Makefile build

When building a library using make, you'll get many kiunds of linking errors and .so file errors, this blog is about few of those kind of errors and how to fix them

I was trying to build OpenCV for Jetson-Nano and this was the kind of error `No rule to make target '<some .so file>', needed by 'lib/<some .so file>'. Stop.`

```shell
make[2]: *** No rule to make target '/usr/lib/libfreetype.so', needed by 'lib/libopencv_freetype.so.4.2.0'.  Stop.
make[2]: *** Waiting for unfinished jobs....
[ 46%] Building CXX object modules/freetype/CMakeFiles/opencv_freetype.dir/src/freetype.cpp.o
[ 46%] Building CXX object modules/features2d/CMakeFiles/opencv_features2d.dir/src/orb.cpp.o
CMakeFiles/Makefile2:3653: recipe for target 'modules/freetype/CMakeFiles/opencv_freetype.dir/all' failed
make[1]: *** [modules/freetype/CMakeFiles/opencv_freetype.dir/all] Error 2
make[1]: *** Waiting for unfinished jobs....
```

This has a simple fix, this happens due to broken symlinks, we just need to fix it.

```shell
nano3@nano3-desktop:~/satyajit/opencv-4.2.0/build$ file /usr/lib/libfreetype.so
/usr/lib/libfreetype.so: broken symbolic link to /usr/lib/x86_64-linux-gnu/libfreetype.so
```

1. Make sure that the library is installed, whichever .so file it is, google it, you'll get like lib-something-dev package needs to be installed, install that first
2. Run `apt-file search`
  ```shell
  nano3@nano3-desktop:~/satyajit/opencv-4.2.0/build$ apt-file search libfreetype.so
  libfreetype6: /usr/lib/aarch64-linux-gnu/libfreetype.so.6
  libfreetype6: /usr/lib/aarch64-linux-gnu/libfreetype.so.6.15.0
  libfreetype6-dev: /usr/lib/aarch64-linux-gnu/libfreetype.so
  ```
3. delete old symlink and create new
  ```shell
  nano3@nano3-desktop:~/satyajit/opencv-4.2.0/build$ sudo rm /usr/lib/libfreetype.so
  [sudo] password for nano3:
  nano3@nano3-desktop:~/satyajit/opencv-4.2.0/build$ ln -s /usr/lib/aarch64-linux-gnu/libfreetype.so /usr/lib/libfreetype.
  so
  ln: failed to create symbolic link '/usr/lib/libfreetype.so': Permission denied
  nano3@nano3-desktop:~/satyajit/opencv-4.2.0/build$ sudo !!
  sudo ln -s /usr/lib/aarch64-linux-gnu/libfreetype.so /usr/lib/libfreetype.so
  ```
4. That's it, now that we have properly linked our `.so` file, run make again !
  ```shell
  make -j4
  ```
 
That's it for now !
