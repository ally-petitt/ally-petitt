+++
title = 'How to Use NVRAM When Emulating an Embedded Device'
date = 2024-07-05
draft = false
showtoc=true
tags= ["Firmware", "Debugging"]
+++


## Introduction
Last week, I came across a piece of firmware that I was particularly curious about. As with typical analysis on such an image, I extracted the squashfs root using binwalk and found the web service, `httpd`, that I was targeting. A smile spread across my face as I typed a command into the terminal to run `httpd` in QEMU user mode, but after pressing the “Enter” key, I saw it:

```bash
$ sudo chroot . ./qemu-armeb-static /usr/sbin/httpd -n
cannot open /dev/nvram
```

Cannot open `/dev/nvram`. This error highlights a somewhat common obstacle when emulating embedded devices. Interacting with NVRAM can be an important step in the expected execution of an application, which can be made even more complicated when emulating a device.


### The Solution
In this article, I’ll walk through one method that I used to hook into the functions that request and set data from `/dev/nvram`, enabling me to intercept and modify interactions with the NVRAM while my target application runs. This allows us to have more granular control of the values returned by calls to the NVRAM so that we can return our desired values. As a result, the application runs smoothly as though an NVRAM device actually existed on the system. Think of it like a man-in-the-middle attack on NVRAM that our target application is oblivious to.

*Note: By "target application" or "target binary", I am referring to the binary that is being emulated in QEMU and relies on the NVRAM. In my specific case, this is `httpd` in my extracted firmware.*

This article aims to walk you through my thought process in approaching this rather than solely offering a copy-paste solution. I have found reading other researcher's thought processes to be helpful in the past and was hoping to pay it forward. As such, my troubleshooting steps and some of my (failed) ideas are included.

**TLDR;** I built an instance of [crosstool-ng](https://github.com/crosstool-ng/crosstool-ng) to create an `armeb-unknown-eabi` toolchain. This was necessary for me to cross-compile [nvram-faker](https://github.com/zcutlip/nvram-faker) in the target architecture: big endian, 32-bit ARM. Nvram-faker allowed us to hijack function calls originally intended for libnvram.so, which is a library used to interface with `/dev/nvram`, through the `LD_PRELOAD` variable. I ended up patching `nvram-faker` in my [own fork](https://github.com/ally-petitt/nvram-faker) to solve dynamic linking issues and solve a bug that caused a segfault.



### Disclaimer

I am just a hobbyist. There are likely more efficient and/or robust ways to add support for NVRAM when emulating firmware with QEMU. This is just one of the solutions that I came across on my own. All the information in the article is accurate to the best of my knowledge and has the chance to be incorrect. I have done my best to fact-check the information presented.

![Picture of NVRAM](https://www.mouser.com/images/cypresssemiconductor/hd/tssop44.jpg)


## What is NVRAM?

NVRAM, or non-volatile RAM, is a type of computer memory that maintains the values stored within it after the computer has been turned off and on again. Often, the NVRAM contains configuration information that can help direct applications during the boot process or runtime. It is almost like a Solid-State Drive (SSD), but with far less storage capacity, increased speed, and it is generally soldered onto the motherboard.

## Setup

The following table shows a high-level overview of the firmware I am dealing with.

| Architecture | ARMv5 (armeb) |
| --- | --- |
| Chipset | IXP425 |
| ABI | EABI5 |
| Operating System | OpenWRT |
| Endianess | Big Endian |

**Goal:** Run the binary `/usr/sbin/httpd` in our firmware image.

**Problem:** `httpd` relies on NVRAM for configuration data. Since we are emulating in QEMU, we do not have the physical NVRAM to provide this data, so `httpd` is unable to initialize.

**Potential Solutions:** 

1. Create a custom kernel module to respond with the expected values at `/dev/nvram`.
    * This can be more complex and would require reverse engineering of `libnvram.so` to understand how `/dev/nvram` expects to be interacted with in order to recreate the appropriate interface. 
    * Building a specific implementation for this firmware may be less portable to other firmware images compared to our 2nd possible solution.
2. Hook function calls to libnvram.so and override their return value.
    * Potentially faster to implement.
    * There is already an open-source project called [nvram-faker](https://github.com/zcutlip/nvram-faker) that can be used as a starting point.

Needless to say, I’ll be exploring solution 2 in this article. Although, I am open to creating the kernel module if it is requested since I can see if having educational value.


## Nvram faker

[Nvram-faker](https://github.com/zcutlip/nvram-faker) is a GitHub project created by Zachary Cutlip in 2013 to "intercept calls to libnvram when running embedded linux applications in emulated environments." Despite its old age, `Nvram-faker` is still very usable for today's use cases.

### How it Works
Our `httpd` binary does not attempt to access `/dev/nvram`, the NVRAM device file in the firmware, directly. Instead, it uses a shared object file called `libnvram.so` in order to abstract interaction with NVRAM. This dependency on `libnvram.so` is evident in the "About" window on `httpd` in Ghidra.

![Picture of linked libraries on the httpd binary](/images/nvram-1.png)
*Yes, my Ghidra is in dark mode.*

The `httpd` program imports functions like `nvram_get()` and `nvram_set()` from `libnvram.so` that manage interactions with `/dev/nvram`. If we were able to set our own definitions for what `nvam_get()` and `nvram_set()` do, we can effectively control the values that `httpd` receives when attempting to interact with the NVRAM through these functions.

That is what `nvram-faker` does. We can recreate the NVRAM-related procedures called by `httpd` in `nvram-faker.c` and compile them into our own library, `libnvram-faker.so`.

Furthermore, `httpd` dynamically links the `libnvram.so` library with the dynamic linker [ld](https://man7.org/linux/man-pages/man8/ld.so.8.html). By setting the environment variable `LD_PRELOAD` to `libnvram-faker.so`, `ld` is instructed to load our crafted shared object file before the real `libnvram.so` library is loaded in. Consequently, the NVRAM-related procedures that we defined in `nvram-faker.c` (and by extension `libnvram-faker.so`) will be called instead of the intended symbols in `libnvram.so`.

`Nvram-faker` is programmed to parse through key-value pairs stored in a `/nvram.ini` file in the firmware squashfs root. When the target binary calls `nvram_get()`, the value defined in `/nvram.ini` for the requested key is returned.

# Building Nvram-Faker
As previously mentioned, the firmware that I am emulating is big-endian, 32-bit ARM. Since `libnvram-faker.so` will be preloaded into a binary in that firmware, it must match that ARM architecture. This means that I will need to cross-compile our `nvram-faker`.

Cross-compiling requires a cross-compilation toolchain. ARM offers plenty of [toolchains](https://developer.arm.com/Tools%20and%20Software/GNU%20Toolchain), however, none of them matched what I was looking for, so I decided to use [crosstool-ng](https://github.com/crosstool-ng/crosstool-ng) to compile my own toolchain. You can read their [documentation](https://crosstool-ng.github.io/docs/) for more information on how this works.

## Making a Cross-Compilation Toolchain with Crosstool-ng
I used the following commands to begin building `crosstool-ng`:

```bash
sudo apt install help2man libtool-bin #dependencies
git clone https://github.com/crosstool-ng/crosstool-ng
cd crosstool-ng
git checkout tags/crosstool-ng-1.26.0
./bootstrap
./configure --prefix="$PWD/build"
make && make install
```

With the created `ct-ng` binary, I began to configure my toolchain. Luckily, there was already a sample configuration from `armeb-unknown-eabi`, which is a match for my target architecture (I was able to tell based on the [toolchain naming conventions](https://stackoverflow.com/questions/5731495/can-anyone-explain-the-gcc-cross-compiler-naming-convention/5731708#5731708)):

```bash
./build/bin/ct-ng show-samples
./build/bin/ct-ng show-armeb-unknown-eabi
./build/bin/ct-ng armeb-unknown-eabi # load it into config
./build/bin/ct-ng menuconfig
```

I verified in the menu configuration that the settings for our target toolchain matched what I was expecting for this compilation in the “Target Options” menu.

![Picture of the "Target Options" menu in the ct-ng menuconfig](/images/nvram-2.png)

I later encountered difficulties with downloading the tarballs needed by `crosstool-ng`. To resolve this, I configured a mirror: [https://ftp.gnu.org/gnu/](https://ftp.gnu.org/gnu/).

![Mirror added to menuconfig](/images/nvram-5.png)


Future me experienced more errors relating to missing necessary C library header files. To address this, I enabled newlib.

![Enabling the newlib C library in the ct-ng menuconfig](/images/nvram-6.png)

We can now begin to build our target toolchain!

```bash
./build/bin/ct-ng build
```

The build process took about 40 minutes and eventually resulted in an error when attempting to compile `gdbserver`:

```bash
[ERROR]    configure: error: C compiler cannot create executables
[CFG  ]    See `config.log' for more details
```

I was worried that this error may have stopped the target toolchain from being created. I searched within the `.build` directory of `crosstool-ng` to check that the target binaries I was looking for were created.

```bash
$ find . -name '*armeb-unknown-eabi*' -type f
./.build/armeb-unknown-eabi/buildtools/share/man/man1/armeb-unknown-eabi-gcc.1
./.build/armeb-unknown-eabi/buildtools/share/man/man1/armeb-unknown-eabi-gcov-tool.1
./.build/armeb-unknown-eabi/buildtools/share/man/man1/armeb-unknown-eabi-gcov.1
./.build/armeb-unknown-eabi/buildtools/share/man/man1/armeb-unknown-eabi-cpp.1
./.build/armeb-unknown-eabi/buildtools/share/man/man1/armeb-unknown-eabi-gcov-dump.1
./.build/armeb-unknown-eabi/buildtools/bin/armeb-unknown-eabi-gcov-tool
./.build/armeb-unknown-eabi/buildtools/bin/armeb-unknown-eabi-gcov
./.build/armeb-unknown-eabi/buildtools/bin/armeb-unknown-eabi-gcc-ranlib
./.build/armeb-unknown-eabi/buildtools/bin/armeb-unknown-eabi-cpp
./.build/armeb-unknown-eabi/buildtools/bin/armeb-unknown-eabi-gcc
./.build/armeb-unknown-eabi/buildtools/bin/armeb-unknown-eabi-gcc-ar
./.build/armeb-unknown-eabi/buildtools/bin/armeb-unknown-eabi-gcc-nm
./.build/armeb-unknown-eabi/buildtools/bin/armeb-unknown-eabi-gcov-dump
./.build/armeb-unknown-eabi/buildtools/bin/armeb-unknown-eabi-gcc-13.2.0

```

Perfect, the tools we need have been built. We can use the gdbserver built into QEMU for debugging.  Let’s begin to compile nvram-faker.

```bash
git clone https://github.com/zcutlip/nvram-faker.git
cd nvram-faker
CC=$HOME/Documents/firmware/crosstool-ng/.build/armeb-unknown-eabi/buildtools/bin/armeb-unknown-eabi-gcc make
```

Unfortunately, attempting to build nvram-faker with `make` resulted in two distinct errors. First, the file `$HOME/Documents/firmware/crosstool-ng/.build/armeb-unknown-eabi/buildtools/lib/gcc/armeb-unknown-eabi/13.2.0/../../../../armeb-unknown-eabi/lib/crt0.o` was not found. 

Let’s investigate.

## Troubleshooting Nvram-Faker Compilation

Since `make` appeared to expect `crt0.o` in a specific location, I figured that changing the search path to include the location that it was actually at would resolve the issue. I began by locating a copy of `crt0.o` that exists

```bash
$ find . -name 'crt0.o'
./.build/armeb-unknown-eabi/build/build-libc/armeb-unknown-eabi/newlib/crt0.o
```

Then, I used `$CFLAGS` such as `-I` and `-L` to include the path `./.build/armeb-unknown-eabi/build/build-libc/armeb-unknown-eabi/newlib/`. I also tried including the `crt0.o` file directly by manually using the `armeb-unkown-eabi-gcc` compiler with `crt0.o` as an input file, but neither of these worked.

I ultimately decided to copy `crt0.o` to the location that the cross-compiler was searching for it. 

```bash
mkdir -p $HOME/Documents/firmware/crosstool-ng/.build/armeb-unknown-eabi/buildtools/lib/gcc/armeb-unknown-eabi/13.2.0/../../../../armeb-unknown-eabi/lib
# the symlink didn't work
#ln -s ./.build/armeb-unknown-eabi/build/build-libc/armeb-unknown-eabi/newlib/crt0.o $HOME/Documents/firmware/crosstool-ng/.build/armeb-unknown-eabi/buildtools/lib/gcc/armeb-unknown-eabi/13.2.0/../../../../armeb-unknown-eabi/lib/crt0.o
cp ./.build/armeb-unknown-eabi/build/build-libc/armeb-unknown-eabi/newlib/crt0.o $HOME/Documents/firmware/crosstool-ng/.build/armeb-unknown-eabi/buildtools/lib/gcc/armeb-unknown-eabi/13.2.0/crt0.o
```

Now that we've solved the first error, let's move on to the next!

```bash
kali@kali:~/Documents/firmware/nvram-faker$ CC=$HOME/Documents/firmware/crosstool-ng/.build/armeb-unknown-eabi/buildtools/bin/armeb-unknown-eabi-gcc make
$HOME/Documents/firmware/crosstool-ng/.build/armeb-unknown-eabi/buildtools/bin/armeb-unknown-eabi-gcc -Wall -I./contrib/inih -ggdb -DINI_MAX_LINE=2000 -DINI_USE_STACK=0 -fPIC -c -o nvram-faker.o nvram-faker.c
nvram-faker.c:1:10: fatal error: stdlib.h: No such file or directory
    1 | #include <stdlib.h>
      |          ^~~~~~~~~~
compilation terminated.
make: *** [Makefile:35: nvram-faker.o] Error 1
```

It seems that `make` was having difficulty with finding our C library. I ended up searching for the `stdlib.h` header file in the `crosstool-ng` directory.

```bash
~/Documents/firmware/crosstool-ng$ find . -name stdlib.h
./.build/armeb-unknown-eabi/build/build-gdb-cross/gnulib/import/stdlib.h
./.build/src/gcc-13.2.0/fixincludes/tests/base/stdlib.h
./.build/src/gcc-13.2.0/fixincludes/tests/base/ansi/stdlib.h
./.build/src/gcc-13.2.0/libstdc++-v3/include/tr1/stdlib.h
./.build/src/gcc-13.2.0/libstdc++-v3/include/c_compatibility/stdlib.h
./.build/src/newlib-4.3.0.20230120/newlib/libc/include/ssp/stdlib.h
./.build/src/newlib-4.3.0.20230120/newlib/libc/include/machine/stdlib.h
./.build/src/newlib-4.3.0.20230120/newlib/libc/include/stdlib.h
./.build/src/newlib-4.3.0.20230120/newlib/libc/machine/powerpc/machine/stdlib.h
```

I first tried using the stdlib from `./.build/armeb-unknown-eabi/build/build-gdb-cross/gnulib/import/`, however, this resulted additional errors. My speculation is that the header files in there were intended for my host architecture, x64, and not the target architecture of 32-bit, big-endian ARM. 

This was the point that I realized that rebuilding with newlib would solve this problem. This enabled me to include `./.build/src/newlib-4.3.0.20230120/newlib/libc/include/` as a search directory in our make command via the CFLAGS `-I` option:

```bash
 make CC=$HOME/Documents/firmware/crosstool-ng/.build/armeb-unknown-eabi/buildtools/bin/armeb-unknown-eabi-gcc CFLAGS="-I$HOME/Documents/firmware/crosstool-ng/.build/src/newlib-4.3.0.20230120/newlib"
```

With that, our problems are solved.

### Compiling nvram-faker.

Now that we are properly set up for building `nvram-faker`, let’s run `make` to create `libnvram-faker.so`.

```bash
$ make CC=$HOME/Documents/firmware/crosstool-ng/.build/armeb-unknown-eabi/buildtools/bin/armeb-unknown-eabi-gcc CFLAGS="-I$HOME/Documents/firmware/crosstool-ng/.build/src/newlib-4.3.0.20230120/newlib"
make: Nothing to be done for 'all'.
$ file ./libnvram-faker.so 
./libnvram-faker.so: ELF 32-bit MSB shared object, ARM, EABI5 version 1 (SYSV), dynamically linked, with debug_info, not stripped
```

The `file` output is exactly what we want as it matches the other binaries executed on the firmware like busybox:

```bash
kali@kali:~/Documents/firmware/_extracted_firmware/squashfs-root$ file ./bin/busybox 
./bin/busybox: ELF 32-bit MSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-musl-armeb.so.1, no section header
```

### Using nvram-faker

Now, I’ll copy the libnvram-faker.so file and re-enter the chroot environment at the base of our firmware image’s squashfs file system.

```bash
cp ../../nvram-faker/libnvram-faker.so .
touch nvram.ini
```

Let's test it!

```bash
$ sudo chroot . ./qemu-armeb-static /bin/sh -c "LD_PRELOAD=/libnvram-faker.so /usr/sbin/httpd"
Error relocating /libnvram-faker.so: initialise_monitor_handles: symbol not found
Error relocating /libnvram-faker.so: __libc_init_array: symbol not found
Error relocating /libnvram-faker.so: main: symbol not found
Error relocating /libnvram-faker.so: __libc_fini_array: symbol not found
Error relocating /libnvram-faker.so: __heap_limit: symbol not found
Error relocating /libnvram-faker.so: _impure_ptr: symbol not found
Error relocating /libnvram-faker.so: _ctype_: symbol not found

```

Hm, it looks like we have some more errors to resolve. Let’s understand why these are happening and address them. Based on the error message, I am going to assume that the linker was expecting the symbols above to be in our library (perhaps as some ABI standard?) and since they were not found, it has difficulty with determining where to place our library in memory. This is just a guess, so feel free to correct me if I’m wrong.

### Troubleshooting libnvram-faker.so

I had two ideas that I thought may resolve this by including the expected symbols:

1. Recompiling `libnvram-faker.so` with our cross-compiled  `ld`, `ar`, `as`, `strip`, and `nm` binaries since I did not include these before.
    - This operates on the assumption that including these binaries would automatically create the expected symbols.
2. Manually patching `nvram-faker.c` to include the symbols that the linker is expecting.

I started out testing the first idea.

```bash
$ make clean
rm *.o
rm *.so
rm nvram_faker_exe
rm: cannot remove 'nvram_faker_exe': No such file or directory
make: [Makefile:46: clean] Error 1 (ignored)
make -C ./contrib/inih clean
make[1]: Entering directory '$HOME/Documents/firmware/nvram-faker/contrib/inih'
rm *.o
make[1]: Leaving directory '$HOME/Documents/firmware/nvram-faker/contrib/inih'

$ make CC=$HOME/Documents/firmware/crosstool-ng/.build/armeb-unknown-eabi/buildtools/bin/armeb-unknown-eabi-gcc CFLAGS="-I$HOME/Documents/firmware/crosstool-ng/.build/src/newlib-4.3.0.20230120/newlib/libc/include" LD=$HOME/x-tools/armeb-unknown-eabi/bin/armeb-unknown-eabi-ld AR=$HOME/x-tools/armeb-unknown-eabi/bin/armeb-unknown-eabi-ar STRIP=$HOME/x-tools/armeb-unknown-eabi/bin/armeb-unknown-eabi-strip NM=$HOME/x-tools/armeb-unknown-eabi/bin/armeb-unknown-eabi-nm 
$HOME/Documents/firmware/crosstool-ng/.build/armeb-unknown-eabi/buildtools/bin/armeb-unknown-eabi-gcc -Wall -I./contrib/inih -I$HOME/Documents/firmware/crosstool-ng/.build/src/newlib-4.3.0.20230120/newlib/libc/include -fPIC -c -o nvram-faker.o nvram-faker.c
make -C ./contrib/inih ini.o
make[1]: Entering directory '$HOME/Documents/firmware/nvram-faker/contrib/inih'
$HOME/Documents/firmware/crosstool-ng/.build/armeb-unknown-eabi/buildtools/bin/armeb-unknown-eabi-gcc -I$HOME/Documents/firmware/crosstool-ng/.build/src/newlib-4.3.0.20230120/newlib/libc/include -fPIC -c -o ini.o ini.c
make[1]: Leaving directory '$HOME/Documents/firmware/nvram-faker/contrib/inih'
cp ./contrib/inih/ini.o .
$HOME/Documents/firmware/crosstool-ng/.build/armeb-unknown-eabi/buildtools/bin/armeb-unknown-eabi-gcc -shared -o libnvram-faker.so nvram-faker.o ini.o -Wl,-nostdlib
```

After copying it back to our squashfs root and trying again, it seems the same issue persists. This rules out incompatible `ld`, `ar`, `as`, `strip`, or `nm` binaries as the root cause. I wasn't confident that this would be the solution, but it was worth a shot. Now I know more.

```bash
$ cp ./libnvram-faker.so ../_extracted_firmware/squashfs-root/
$ sudo chroot . ./qemu-armeb-static /bin/sh -c "LD_PRELOAD=/libnvram-faker.so /usr/sbin/httpd"
Error relocating /libnvram-faker.so: initialise_monitor_handles: symbol not found
Error relocating /libnvram-faker.so: __libc_init_array: symbol not found
Error relocating /libnvram-faker.so: main: symbol not found
Error relocating /libnvram-faker.so: __libc_fini_array: symbol not found
Error relocating /libnvram-faker.so: __heap_limit: symbol not found
Error relocating /libnvram-faker.so: _impure_ptr: symbol not found
Error relocating /libnvram-faker.so: _ctype_: symbol not found
```

Instead, let’s attempt the second idea. I created [my own fork](https://github.com/ally-petitt/nvram-faker) of nvram-faker and got to work.

As I was creating the patch to manually add the expected symbols into `nvram-faker.c`, I made the observation that many of these symbols were defined in common C libraries like `ctypes.h` and `stdlib.h`. This made me suspect that the issue might reside in the flags relating to the C library. 

I quickly tried recompiling without the `-nostdlib` flag by modifying the `nvram-faker` `Makefile` and recompiling with the `-static` flag. Neither of these worked, so I continued with the original plan, which involved appending the following to `nvram-faker.c` and recompiling:

```bash
void main(){
	write(1, "libnvram-faker.so has loaded\n", 29);
}

void* initialise_monitor_handles;
void* __heap_limit;
void (*__libc_fini_array[]) (void);
void (*__libc_init_array[]) (void);
struct _reent * _impure_ptr;
char _ctype_;
```

I discovered the data types that work through a combination of compiler warnings and grepping for definitions such as:

```bash
kali@kali:~/Documents/firmware/crosstool-ng/.build/src/newlib-4.3.0.20230120/newlib/libc/include$ grep -r "_ctype_" .
./ctype.h:extern	__IMPORT const char	_ctype_[];
```

Let’s try to run this again!

```bash
kali@kali:~/Documents/firmware/_extracted_firmware/squashfs-root$ sudo chroot . ./qemu-armeb-static /bin/sh -c "LD_PRELOAD=/libnvram-faker.so /usr/sbin/httpd"
cannot open /dev/nvram
httpd : httpd cannot start. ssl and/or http must be selected
```

Well, our library loaded successfully this time, but it’s not quite what we were hoping for. From my prior reverse engineering of interesting libraries in this firmware, I knew that the error message we received comes from `libnvram.so`, which we can verify with grep.

```bash
$ grep -r "cannot open /dev/nvram" .
Binary file ./lib/libnvram.so matches
```

### Debugging Libnvram-faker.so

I wanted to get to the root of why this error message was still appearing despite using `libnvram-faker.so` to hook NVRAM-related functionality.

#### Attempt 1: GDB

I first attempted to debug using GDB. In QEMU, I used the built-in GDB server to listen on port 1234.

```bash
sudo chroot . ./qemu-armeb-static -g 1234 -E LD_PRELOAD=/libnvram-faker.so /usr/sbin/httpd
```

In another terminal window I modified my `.gdbinit` file to expedite the process of connecting and debugging to our target application.

```bash
$ cat ~/.gdbinit
set disassembly-flavor intel
set print asm-demangle on
source $HOME/Downloads/pwndbg/gdbinit.py
set arch arm
set follow-fork-mode child
target extended-remote :1234
```

Then, I connected to the remote process.

```bash
gdb-multiarch ./usr/sbin/httpd
```

![Pwndbg context output after connecting to the httpd debugging process](/images/nvram-7.png)

To make debugging easier, I calculated the offset of the memory addresses in our debugging process to memory addresses in the `httpd` binary itself. When I ran the `load` command in GDB with this offset as the second parameter, it made the addresses in the debugger consistent with those in my Ghidra CodeBrowser.

To make this calculation, I subtracted PC (the ARM [program counter register](https://developer.arm.com/documentation/107656/0101/Registers/Registers-in-the-register-bank/R15--Program-Counter--PC-)) with the address of the first entry instruction in `httpd` to adjust our addresses to match that of the binary. 

> {{< collapse summary="**Note on verifying offset**" >}}
I had verified that we were at the first entry instruction by matching the byte sequence that our PC was at (`x/3wx $PC`) to the hex dump of the `httpd` entrypoint (made easier through the use of a Ghidra search functionality).
{{</ collapse >}}

```bash
pwndbg> load ./usr/sbin/httpd 4286266732
```

This is where the problems with GDB began. There appeared to be an issue with the timing of the program between when I connected to the debugging process and it began execution. If the delay was more than a few milliseconds, the `libc_start_main` function would have a segmentation fault. I tried automating loading the program and setting breakpoints in `.gdbinit`, but even this was not fast enough to prevent a segfault. 

I don’t know if there was a race condition or something else preventing normal execution. If anyone knows why this may have happened, please do let me know since I am quite puzzled by it.

I asked myself the question, “Is there another way to do this?”. I realized what I was really looking for was a call stack for when that error message was printed. I got an idea of how I could recreate this through static analysis.

#### Attempt 2: Ghidra

I started by pinpointing where our error message appears and checking its implementation in our fake NVRAM library. 

In Ghidra, I opened a project with `libnvram.so` loaded in the CodeBrowser. Using the built-in tool `Window > Defined Strings`, I was able to search for our error message, “`cannot open /dev/nvram`."

![Ghidra defined strings window showing our error message](/images/nvram-8.png)

I followed the location of this string in the binary to reveal that it is referenced in a procedure that Ghidra called `FUN_00012420`.

![The error message in the disassembly of the binary](/images/nvram-9.png)

This function is called by some of the major functions in `libnvram.so` such as `nvram_get`.

![The call tree of FUN_00012420](/images/nvram-10.png)

When quickly listing the NVRAM functions imported in `httpd`(which there are defined strings for), it does not appear that `httpd` is calling a libnvram function that would trigger our error. However, that doesn’t mean that a call to this endpoint isn’t abstracted some other way.

![Defined strings with "nvram" in them in `httpd`](/images/nvram-11.png)

I decided to analyze a complete call graph of `FUN_00012420` (as opposed to the first layer of the call tree), which confirmed my suspicions: `nvram_set` called a function that called `FUN_00012420`. This means that `nvram_set`, a function called by `httpd`, was very likely the entrypoint to our “`cannot find /dev/nvram`” error message.

![Call graph of FUN_00012420](/images/nvram-12.png)

As stated in the `nvram-faker` [README](https://github.com/zcutlip/nvram-faker?tab=readme-ov-file#using):

> The library currently does not support calls to `nvram_set()`.

My hypothesis was that since `nvram_set()` was not defined in the `libnvram-faker.so` library that we created, it was being imported from `libnvram.so`. In turn, `FUN_00012420` is called which attempts to open `/dev/nvram` and fails (since the kernel module for `/dev/nvram` does not exist), printing our error message. 

To verify that this is the case, I once again patched `nvram-faker.c`, but this time I implemented a preliminary definition for `nvram_set` and observed whether the error from before is still present.

The following is what I added to `nvram-faker.c`:

```bash
char * nvram_set(char * key, char * value) {
	write(1, "nvram_set called\n", 17);
}
```

After I compiled and ran `httpd`, this was my output:

```bash
$ sudo chroot . ./qemu-armeb-static -E LD_PRELOAD=/libnvram-faker.so /usr/sbin/httpd
nvram_set called
httpd : httpd cannot start. ssl and/or http must be selected
```

Perfect! The previous error that `/dev/nvram` was not found is no longer appearing and in its place is the output that we injected for the `nvram_set()` function I defined. This verifies that `nvram_set` was the source of the error messages. Now, we can create our own implementation of `nvram_set` to enable full functionality of `httpd.`

To streamline the development process, I made my own script to clean, build, and test the library:

```bash
#!/bin/bash

set -e # exit on failure

if [ $# -ne 4 ]; then
  echo "Usage: $0 <compilation_toolchain_binaries_path> <squashfs_root_path> <newlib_path> <target_binary_path>"
  echo "Example: $0 $HOME/x-tools/armeb-unknown-eabi/bin $HOME/_extracted_firmware/squashfs-root $HOME/Documents/firmware/crosstool-ng/.build/src/newlib-4.3.0.20230120/newlib/libc/include /usr/sbin/httpd"
  exit 1
fi

TOOLCHAIN_PATH=$1
SQUASHFS_PATH=$2
NEWLIB_PATH=$3
TARGET_BINARY=$4

make clean
make CC="${TOOLCHAIN_PATH}/armeb-unknown-eabi-gcc" CFLAGS="-DDEBUG=1 -I${NEWLIB_PATH}" LD="${TOOLCHAIN_PATH}/armeb-unknown-eabi-ld AR=${TOOLCHAIN_PATH}/armeb-unknown-eabi-ar STRIP=${TOOLCHAIN_PATH}/armeb-unknown-eabi-strip NM=${TOOLCHAIN_PATH}/armeb-unknown-eabi-nm"

cp ./libnvram-faker.so "${SQUASHFS_PATH}"
sudo chroot "${SQUASHFS_PATH}" ./qemu-armeb-static -E LD_PRELOAD=/libnvram-faker.so "${TARGET_BINARY}"

```

I encountered a segmentation fault when debug was enabled due to how `fprintf` was being used to log debug output through the `DEBUG_PRINTF` macro. I ended up redefining `DEBUG_PRINTF` and `LOG_PRINTF` to be based on `printf` instead of `fprintf` to prevent the segfault.


> {{< collapse summary="**Expand to learn why fprintf was originally used instead of printf**" >}}
After a bit of digging, I would discover that `fprintf()` was used for debug and log output in the original `nvram-faker` repository as a workaround to how `stdout` was being redirected after `system()` was called in the webserver it was used to emulate. As explained in the [author’s blogpost](https://shadowfile.inode.link/blog/2015/01/patching-emulating-and-debugging-a-netgear-embedded-web-server/), `fprintf()` allowed him to print to `stderr`, which was visible in the console, unlike the output of `printf()` that goes to `stdout` and was not visible after `system()` was called.
{{</ collapse >}}

### Adding Support for nvram_set

It was originally my intention to add `nvram_set()` support to `nvram-faker.c` to fully mimic a real NVRAM interaction, however, an event came up in my personal life that delayed this. I may endeavor to do so in the future when the timing is better, however, for now I have opted to simply create my [own fork](https://github.com/ally-petitt/nvram-faker) with the following improvements on the old one:

- More informative debug output
- Resolve segmentation fault in debugging output
- Added additional symbols to `nvram-faker.c` such as `initialise_monitor_handles` and `__libc_fini_array` to avoid linking errors
- Add a preliminary `nvram_set()` function that logs debug output to prevent function calls to `libnvram.so`
- Add a `build_and_run.sh` script to streamline development and testing

## Next Steps
From here, you can simply create and populate an `nvram.ini` file at the root of your extracted firmware filesystem and add the keys-value pairs that your target application expects! There are many ways to figure out these expected values. For instance, you can run the target application and view the NVRAM values it requests in the debugging output. You can also reverse engineer the target application to see the values it expects.

Here is an excerpt from the `nvram-faker` example [nvram.ini](https://github.com/zcutlip/nvram-faker/blob/master/nvram.ini) file for reference:

```
[config]
os_name=linux
os_version=1
upnp_port=9999
upnp_ad_time=30
upnp_sub_timeout=60
upnp_conn_retries=10
log_level=10
lan_hwaddr=52:54:00:12:34:58
lan_ifname=eth0
lan1_ifname=wlan0
```


# Conclusion
This article was quite lengthy and technical compared to what I normally put out. It started as me wanting to test out a chroot sandbox on a router and escalated into emulating a webserver and needing a way to interface with NVRAM. This was a great exercise that taught me more about cross-compilation, NVRAM, and general troubleshooting. One year ago, I would have been overwhelmed by the amount of errors that I encountered while getting this to work (this article detailed less than half of them for brevity), so working through this helped to strengthen my perseverance as a researcher.

Overall, this was an interesting rabbit hole to go down. I hope it made as much sense in the text as it did in my head. If any details here are inaccurate, please feel free to contact me and I will be happy to correct the information in this article.

Thanks for reading!

