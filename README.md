# DARPA Challenge Binaries on Linux, OS X, and Windows

[![Build Status](https://img.shields.io/github/workflow/status/trailofbits/cb-multios/CI/master)](https://github.com/trailofbits/cb-multios/actions?query=workflow%3ACI)
[![Slack Status](https://empireslacking.herokuapp.com/badge.svg)](https://empireslacking.herokuapp.com)

The DARPA Challenge Binaries (CBs) are custom-made programs specifically designed to contain vulnerabilities that represent a wide variety of crashing software flaws. They are more than simple test cases, they approximate real software with enough complexity to stress both manual and automated vulnerability discovery. The CBs come with extensive functionality tests, triggers for introduced bugs, patches, and performance monitoring tools, enabling benchmarking of patching tools and bug mitigation strategies.

The CBs were originally developed for DECREE -- a custom Linux-derived operating system that has no signals, no shared memory, no threads, no standard libc runtime, and only seven system calls -- making them incompatible with most existing analysis tools. In this repository, we have modified the CBs to work on Linux and OS X by replacing the build system and re-implementing CGC system calls via standard libc functionality and native operating system semantics. Scripts have been provided that help modify the CBs to support other operating systems.

The CBs are the best available benchmark to evaluate program analysis tools. Using them, it is possible to make comparisons such as:

* How good are tools from the Cyber Grand Challenge vs. existing program analysis and bug finding tools?
* When a new tool is released, how does it stack up against the current best?
* Do static analysis tools that work with source code find more bugs than dynamic analysis tools that work with binaries?
* Are tools written for Mac OS X better than tools written for Linux, and are they better than tools written for Windows?

## Components

### challenges
This directory contains all of the source code for the challenge binaries. Challenges that are not building or are not yet supported are in the `disabled-challenges` directory.

### include
This directory contains `libcgc`, which implements the syscalls to work on non-DECREE systems. `libcgc` currently works on OS X and Linux.

### tools
This folder contains Python scripts that help with modifying, building, and testing the original challenges.

#### tester.py
This is a helper script to test all challenges using `cb-test`. Results are summarized and can be output to an excel spreadsheet. More details in the [testing section](#testing) below.

## Building

The following steps will build both the patched and unpatched binaries in `build/challenges/[challenge]/`.

### MacOS

The challenges build as i386 binaries, but Mac OS 10.14+ only supports building x86-64 binaries by default. To enable i386 support, run the following command:

```
sudo installer -pkg /Library/Developer/CommandLineTools/Packages/macOS_SDK_headers_for_macOS_10.14.pkg -target /
```

After this, proceed to the common directions for MacOS and Linux.

### Linux

The following packages are required for building the challenges on Linux:

```
libc6-dev libc6-dev-i386 gcc-multilib g++-multilib clang cmake
```

### MacOS/Linux Common Directions

First, install pre-requisites via pip.

```
sudo pip install xlsxwriter pycrypto defusedxml pyyaml matplotlib
```

Then to build all challenges, run:

```bash
$ ./build.sh
```

If you are **absolutely certain** that you don't intend to use any of the Python components of the
build or repository, you can tell the build script to ignore them:

```bash
$ NO_PYTHON_I_KNOW_WHAT_I_AM_DOING_I_SWEAR=1 ./build.sh
```

This is **not** a publicly supported build mode.

### Windows

The following packages are required for building the challenges on Windows:

|Name                             |  Version/Info                       |
|---------------------------------|-------------------------------------|
| Visual Studio Build Tools       | included in Visual Studio           |
| Windows SDK (for running tests) | optional install with Visual Studio |
| CMake                           | 3.1+                                |
| Clang                           | 3.8+                                |

**Note:** depending on where you clone the repo, you may run into build errors about the path being too long. It's best to clone the repo closer to your root directory, e.g. `C:\cb-multios\`

To build all challenges, run:

```
> powershell .\build.ps1
```

## Testing

The `tester.py` utility is a wrapper around `cb-test` that can be used to test challenges and summarize results. The [`cb-test`](https://github.com/CyberGrandChallenge/cb-testing) tool is a testing utility created for the DARPA Cyber Grand Challenge to verify CBs are fully functional.

`cb-test` has been modified to run tests locally with no networking involved. All changes include:

* The removal of any kind of server for launching challenges
* Skipping any checks that verify the file is a valid DECREE binary
* Lessening sleeps and timeouts to allow tests to run at a reasonable rate

### Options

```
-a / --all: Run tests against all challenges
-c / --chals [CHALS ...]: Run tests against individual challenges
--povs: Only test POVs for every challenge
--polls: Only test POLLs for every challenge
-o / --output OUTPUT: Output a summary of the results to an excel spreadsheet
```

### Example Usage

The following will run tests against all challenges in `challenges` and save the results to `out.xlsx`:

```bash
$ ./tester.py -a -o out.xlsx
```

To run tests against only two challenges, do this:

```bash
$ ./tester.py -c Palindrome basic_messaging
```

To test all POVs and save the results, run:

```bash
$ ./tester.py -a --povs -o out.xlsx
```

### Types of Tests

All tests are a series of input strings and expected output for a challenge. There are two types of tests that are used:

`POV (Proof of Vulnerability)`: These tests are intended to exploit any vulnerabilities that exist in a challenge. They are expected to pass with the patched versions of the challenges, and in many cases cause the unpatched version to crash.

`POLL`: These tests are used to check that a challenge is functioning correctly, and are expected to pass with both the unpatched and patched versions of challenges.

### Type 1 POV notice

Verifying type 1 POVs relies on analyzing the core dump generated when a process crashes. They can be enabled with:

###### OS X:
```bash
$ sudo sysctl -w kern.coredump=1
```

###### Linux:
```bash
$ ulimit -c unlimited
```

###### Windows:
Merge `tools/win_enable_dumps.reg` into your registry. Note that this will disable the Windows Error Reporting dialog when a program crashes, so it's recommended that you do this in a VM if you want to keep that enabled.

## Current Status

Porting the Challenge Binaries is a work in progress, and the current status of the porting effort is tracked in the following spreadsheet:

https://docs.google.com/spreadsheets/d/1Z2pinCkOqe1exzpvFgwSG2wH3Z09LP9VJk0bm_5jPe4/edit?usp=sharing

## Notes

We use the CMake build system to enable portability across different compilers and operating systems. CMake works across a large matrix of compiler and operating system versions, while providing a consistent interface to check for dependencies and build software projects.

We are working to make this repository easier to use for the evaluation of program analysis tools. If you have questions about the challenge binaries, please [join our Slack](https://empireslacking.herokuapp.com) and we'll be happy to answer them.

## Authors

Porting work was completed by Kareem El-Faramawi and Loren Maggiore, with help from Artem Dinaburg, Peter Goodman, Ryan Stortz, and Jay Little. Challenges were originally created by NARF Industries, Kaprica Security, Chris Eagle, Lunge Technology, Cromulence, West Point Military Academy, Thought Networks, and Air Force Research Labs while under contract for the DARPA Cyber Grand Challenge.

## 关于编译的问题

修改了根目录下`CMakeList.txt`中的如下内容(为了构建统一的程序名称)，具体的见根目录下`CMakeList.txt`的diff情况。

### 静态与动态链接

修改文件`build.sh`中的line 32，其中SHARED为动态链接，STATIC为静态链接
```
LINK=${LINK:-SHARED}
case $LINK in
    SHARED) CMAKE_OPTS="$CMAKE_OPTS -DBUILD_SHARED_LIBS=ON -DBUILD_STATIC_LIBS=OFF";;
    STATIC) CMAKE_OPTS="$CMAKE_OPTS -DBUILD_SHARED_LIBS=OFF -DBUILD_STATIC_LIBS=ON";;
esac
```

对于动态编译的程序，运行时需要将所需的.so库(生成位置一般在根目录/include下或者程序目录下)放入/usr/lib中以便程序能够加载使用。

#### 注意

原本的静态链接选项会进行完全的静态链接，也就是将libc库函数的代码也静态链接进去；这里对根目录下`CMakeList.txt`中的line 71 添加了`-Wl,-Bdynamic -lc`使得程序静态链接libcgc，动态链接libc，生成程序测试:
```
sec@funny:~/workspace/cb-multios/build/challenges/3D_Image_Toolkit$ ldd ./CROMU_00078
	linux-gate.so.1 =>  (0xf7f8b000)
	libc.so.6 => /lib32/libc.so.6 (0xf7dbb000)
	/usr/lib/libc.so.1 => /lib/ld-linux.so.2 (0xf7f8d000)
```

#### 链接器问题(已解决，此处只作记录)

动态链接的程序在运行时可能会出现下面问题

```
sec@funny:~/Desktop/cb-multios/build/challenges/A_Game_of_Chance$ ./NRFIN_00072_1
bash: ./NRFIN_00072_1: No such file or directory
```

问题的原因是程序的链接器不存在，可以使用file查看程序需要的链接器

```
sec@funny:~/Desktop/cb-multios/build/challenges/A_Game_of_Chance$ file NRFIN_00072_1
NRFIN_00072_1: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /usr/lib/libc.so.1, for GNU/Linux 2.6.32, BuildID[sha1]=b8365e95cb3c8cf6411b1487da00d72fdc77690b, not stripped
```

可以看到程序需要的链接器是`/usr/lib/libc.so.1`，但是这个在系统中并不存在。使用ldd查看：

```
sec@funny:~/Desktop/cb-multios/build/challenges/A_Game_of_Chance$ ldd ./NRFIN_00072_1
	linux-gate.so.1 =>  (0xf7efd000)
	libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xf7d29000)
	/usr/lib/libc.so.1 => /lib/ld-linux.so.2 (0xf7eff000)
```

`/usr/lib/libc.so.1`是ld默认的i386链接器路径，算是工具的一个问题，解决方法是在`CMakeList.txt`中修改line72如下：

```
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -static -Wl,-dynamic-linker /lib/ld-linux.so.2 -Wl,-Bdynamic -lc -Wl,--allow-multiple-definition ")
```



### 32位与64位程序

需要修改的地方在根目录`CMakeList.txt`的line 56-65,将其中的-m32编译选项去掉(作者代码中的注释意思好像是不完全支持64位)，如下:

```
    add_compile_options(
        -fno-builtin
        -w
        -g3
    )

    # Link everything 32-bit (until we have a 64-bit option)
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS}")
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS}")
```

初除此之外，由于前面链接器使用的i386，需要将其修改为x64的，也就是`CMakeList.txt`中修改line72如下：

```
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -static -Wl,-dynamic-linker /lib64/ld-linux-x86-64.so.2 -Wl,-Bdynamic -lc -Wl,--allow-multiple-definition ")
```

生成64位程序时可能会出现如下几种错误:

- 强制类型转换错误(比如pointer转为uint32_t)
  - 目前最简单的处理方式就是跳过此程序:找到程序名称，在根目录下`./exclude/linux.txt`中添加进去程序名称
  - 错误程序有：CML/Blubber/Messaging/FailAV
- SSE相关的错误
  - 似乎是浮点数相关的错误，涉及程序：PTaaS/ValveChecks/Charter
  - 目前的方法是找到build目录(或者原目录)下对应程序文件夹中的`CMakeList.txt`,将`-mno-sse`去掉;或者跳过程序？
```
fatal error: error in backend: SSE register return with SSE disabled
clang: error: clang frontend command failed with exit code 70 (use -v to see invocation)
```
- 其他错误
  - FUN:error: initializer element is not a compile-time constant

以上这些只是生成过程中表现出来的问题，还有一些程序能通过编译，但是后面可能会在运行时出现内存错误，具体没有细究。



**编译后二进制文件位于binaries-dynamic-libc文件夹下。**
