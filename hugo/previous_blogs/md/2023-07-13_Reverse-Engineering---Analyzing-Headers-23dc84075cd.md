Reverse Engineering — Analyzing Headers
=======================================

`objdump` is a command line tool that can be used to gain insight into an executable binary. In this article, the tool will be used to dump all of the headers of the ELF binary `heapedit` with the command below. Then, we will analyze and explain each section of output from the top to the bottom.


```
objdump -x ./heapedit
```
Executable and Linkable Format (ELF) files are a common file format for object files, executable binaries, core dumps, and shared libraries. It provides a standardized format for storing executable and object code, symbol information, and other metadata necessary for proper software execution. It may be helpful to first get an overview of an ELF file structure which can be seen in this image:

<img class="graf-image" data-height="1131" data-image-id="0*1ro_BkQGlcyGrJmG.png" data-is-featured="true" data-width="1600" src="https://cdn-images-1.medium.com/max/800/0*1ro_BkQGlcyGrJmG.png"/>

<figcaption class="imageCaption"><a class="markup--anchor markup--figure-anchor" data-href="https://upload.wikimedia.org/wikipedia/commons/e/e4/ELF_Executable_and_Linkable_Format_diagram_by_Ange_Albertini.png" href="https://upload.wikimedia.org/wikipedia/commons/e/e4/ELF_Executable_and_Linkable_Format_diagram_by_Ange_Albertini.png" rel="nofollow noopener" target="_blank">https://upload.wikimedia.org/wikipedia/commons/e/e4/ELF_Executable_and_Linkable_Format_diagram_by_Ange_Albertini.png</a></figcaption>

File Header
-----------

The first segment of output displays information from the file header including its file format, architecture, and flags. I’ll sequentially describe what each section means.


```
heapedit: file format elf64-x86-64  
heapedit  
architecture: i386:x86-64, flags 0x00000112:  
EXEC\_P, HAS\_SYMS, D\_PAGED  
start address 0x0000000000400720
```
* **file format-** `elf64-x86–64` refers to an ELF file that was designed for 64-bit systems and compiled for the x86–64 architecture.
* **architecture**- The architecture `i386:x86–64` indicates that the program is compatible with the i386 and x86–64 architecture.
* **flags 0x00000112**- These flags are Binary File Descriptors ([BFDs](https://sourceware.org/binutils/docs-2.23.1/bfd/index.html)). They come from the `binutils` package which is built into `objdump` and their meaning can be found in the documentation [here](http://sourceware.org/binutils/docs-2.23.1/bfd/BFD-front-end.html#BFD-front-end). In this example, `EXEC_P` means the program is directly executable, `HAS_SYMS` means that the program has a symbol table which helps with debugging, and `D_PAGED` means that the program’s memory is dynamically paged.
* **start address**- The memory address at which the `.text` section begins, which contains the assembly code of the program.

This information is obtained by analyzing the raw bytes at the beginning of the file. The specifications for which bits correspond with which pieces of information can be seen [here](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format).

Program Headers
---------------

The program header (Phdr) is a section that contains necessary information for executing the binary.


```
Program Header:  
 PHDR off 0x0000000000000040 vaddr 0x0000000000400040 paddr 0x0000000000400040 align 2**3  
 filesz 0x00000000000001f8 memsz 0x00000000000001f8 flags r--  
 INTERP off 0x0000000000000238 vaddr 0x0000000000400238 paddr 0x0000000000400238 align 2**0  
 filesz 0x000000000000001c memsz 0x000000000000001c flags r--  
 LOAD off 0x0000000000000000 vaddr 0x0000000000400000 paddr 0x0000000000400000 align 2**21  
 filesz 0x0000000000000c98 memsz 0x0000000000000c98 flags r-x  
 LOAD off 0x0000000000000e00 vaddr 0x0000000000600e00 paddr 0x0000000000600e00 align 2**21  
 filesz 0x0000000000000278 memsz 0x0000000000000288 flags rw-  
 DYNAMIC off 0x0000000000000e10 vaddr 0x0000000000600e10 paddr 0x0000000000600e10 align 2**3  
 filesz 0x00000000000001e0 memsz 0x00000000000001e0 flags rw-  
 NOTE off 0x0000000000000254 vaddr 0x0000000000400254 paddr 0x0000000000400254 align 2**2  
 filesz 0x0000000000000044 memsz 0x0000000000000044 flags r--  
EH\_FRAME off 0x0000000000000b58 vaddr 0x0000000000400b58 paddr 0x0000000000400b58 align 2**2  
 filesz 0x000000000000003c memsz 0x000000000000003c flags r--  
 STACK off 0x0000000000000000 vaddr 0x0000000000000000 paddr 0x0000000000000000 align 2**4  
 filesz 0x0000000000000000 memsz 0x0000000000000000 flags rw-  
 RELRO off 0x0000000000000e00 vaddr 0x0000000000600e00 paddr 0x0000000000600e00 align 2**0  
 filesz 0x0000000000000200 memsz 0x0000000000000200 flags r--
```
Referencing the EL[F man page](https://man7.org/linux/man-pages/man5/elf.5.html), the paraphrased meaning of the different program types (PTs) are below.

* **PHDR**- specifies the location and size of the program header table itself.
* **INTERP**- specifies the location and size of the program interpreter (dynamic linker) path.
* **LOAD**- specifies the location and size of a loadable segment in the binary.
* **DYNAMIC**- specifies the location and size of the dynamic linking information.
* **NOTE**: specifies the location and size of the ELF note segment (ElfN\_Nhdr).
* **EH\_FRAME**: specifies the location and size of the exception handling frame information.
* **STACK**: represents the stack segment, but in this case, it has a size of 0, indicating that it doesn’t occupy any space in the binary.
* **RELRO**: specifies the location and size of the Relocation Read-Only (RELRO) area.

Each program type has its corresponding attributes:

* **off**- offset from beginning of file to first byte of the segment
* **vaddr**- the virtual address that the first byte of the segment resides in memory
* **paddr**- the physical memory address of the first byte of the segment if relevant
* **filesz**- holds the number of bytes of the file image of the segment
* **memsz**- holds the number of bytes of the memory image of hte segment
* **flags**- holds a bitmask of flags describing the read, write, and execute permissions of the segment.
* **align**- specifies the desired alignment of the segment or section in memory. It indicates the power of two that should be used as the alignment constraint.

This table gives an overview of the different sections in memory and where they can be found.

Dynamic Section
---------------

This section contains information about the dynamic linking and runtime symbol resolution of the program. This allows for external libraries to be loaded from disk into memory during the execution of the program.


```
Dynamic Section:  
 NEEDED libc.so.6  
 RUNPATH ./  
 INIT 0x0000000000400650  
 FINI 0x0000000000400af4  
 INIT\_ARRAY 0x0000000000600e00  
 INIT\_ARRAYSZ 0x0000000000000008  
 FINI\_ARRAY 0x0000000000600e08  
 FINI\_ARRAYSZ 0x0000000000000008  
 GNU\_HASH 0x0000000000400298  
 STRTAB 0x0000000000400410  
 SYMTAB 0x00000000004002c0  
 STRSZ 0x00000000000000a8  
 SYMENT 0x0000000000000018  
 DEBUG 0x0000000000000000  
 PLTGOT 0x0000000000601000  
 PLTRELSZ 0x00000000000000f0  
 PLTREL 0x0000000000000007  
 JMPREL 0x0000000000400560  
 RELA 0x0000000000400518  
 RELASZ 0x0000000000000048  
 RELAENT 0x0000000000000018  
 VERNEED 0x00000000004004d8  
 VERNEEDNUM 0x0000000000000001  
 VERSYM 0x00000000004004b8
```
In this example, the file `libc.so.6` needs to be dynamically linked to the program. The ELF man page defines the meaning of the categories on the left column. The right column contains the memory address to store the sections indicated by the left column.

For example, the ELF man page defines `PLTGOT`:


> **DT\_PLTGOT** Address of PLT and/or GOT

In the above code block, the address of the Procedure Linkage Table (PLT) and/or Global Offset Table (GOT) table would be stored at the memory address `0x0000000000601000` as shown by the fact that they are in the same row and the values correspond.

Version References
------------------

This section lists the versions of the dynamically linked libraries that are required for the program to run.

These versions are determined by the linker and requiring them allows for the program to run properly since the library versions used will be compatible with the Application Binary Interface (ABI).


```
Version References:  
 required from libc.so.6:  
 0x0d696917 0x00 04 GLIBC\_2.7  
 0x0d696914 0x00 03 GLIBC\_2.4  
 0x09691a75 0x00 02 GLIBC\_2.2.5
```
In this example, the output indicates that the binary depends on 3 different GLIBC versions: 2.7, 2.4, and 2.2.5.

Sections
--------

This segment of output contains information about the memory location and names of various different sections of the program. This information is useful to the linker since it helps with symbol resolution, relocation to ensure that they reference the correct addresses in memory, and more such as pointing to the program’s initialization and finalization routines.


```
Sections:  
Idx Name Size VMA LMA File off Algn  
 0 .interp 0000001c 0000000000400238 0000000000400238 00000238 2**0  
 CONTENTS, ALLOC, LOAD, READONLY, DATA  
 1 .note.ABI-tag 00000020 0000000000400254 0000000000400254 00000254 2**2  
 CONTENTS, ALLOC, LOAD, READONLY, DATA  
 2 .note.gnu.build-id 00000024 0000000000400274 0000000000400274 00000274 2**2  
 CONTENTS, ALLOC, LOAD, READONLY, DATA  
 3 .gnu.hash 00000024 0000000000400298 0000000000400298 00000298 2**3  
 CONTENTS, ALLOC, LOAD, READONLY, DATA  
 4 .dynsym 00000150 00000000004002c0 00000000004002c0 000002c0 2**3  
 CONTENTS, ALLOC, LOAD, READONLY, DATA  
 5 .dynstr 000000a8 0000000000400410 0000000000400410 00000410 2**0  
 CONTENTS, ALLOC, LOAD, READONLY, DATA  
 6 .gnu.version 0000001c 00000000004004b8 00000000004004b8 000004b8 2**1  
 CONTENTS, ALLOC, LOAD, READONLY, DATA  
 7 .gnu.version\_r 00000040 00000000004004d8 00000000004004d8 000004d8 2**3  
 CONTENTS, ALLOC, LOAD, READONLY, DATA  
 8 .rela.dyn 00000048 0000000000400518 0000000000400518 00000518 2**3  
 CONTENTS, ALLOC, LOAD, READONLY, DATA  
 9 .rela.plt 000000f0 0000000000400560 0000000000400560 00000560 2**3  
 CONTENTS, ALLOC, LOAD, READONLY, DATA  
 10 .init 00000017 0000000000400650 0000000000400650 00000650 2**2  
 CONTENTS, ALLOC, LOAD, READONLY, CODE  
 11 .plt 000000b0 0000000000400670 0000000000400670 00000670 2**4  
 CONTENTS, ALLOC, LOAD, READONLY, CODE  
 12 .text 000003d2 0000000000400720 0000000000400720 00000720 2**4  
 CONTENTS, ALLOC, LOAD, READONLY, CODE  
 13 .fini 00000009 0000000000400af4 0000000000400af4 00000af4 2**2  
 CONTENTS, ALLOC, LOAD, READONLY, CODE  
 14 .rodata 00000057 0000000000400b00 0000000000400b00 00000b00 2**3  
 CONTENTS, ALLOC, LOAD, READONLY, DATA  
 15 .eh\_frame\_hdr 0000003c 0000000000400b58 0000000000400b58 00000b58 2**2  
 CONTENTS, ALLOC, LOAD, READONLY, DATA  
 16 .eh\_frame 00000100 0000000000400b98 0000000000400b98 00000b98 2**3  
 CONTENTS, ALLOC, LOAD, READONLY, DATA  
 17 .init\_array 00000008 0000000000600e00 0000000000600e00 00000e00 2**3  
 CONTENTS, ALLOC, LOAD, DATA  
 18 .fini\_array 00000008 0000000000600e08 0000000000600e08 00000e08 2**3  
 CONTENTS, ALLOC, LOAD, DATA  
 19 .dynamic 000001e0 0000000000600e10 0000000000600e10 00000e10 2**3  
 CONTENTS, ALLOC, LOAD, DATA  
 20 .got 00000010 0000000000600ff0 0000000000600ff0 00000ff0 2**3  
 CONTENTS, ALLOC, LOAD, DATA  
 21 .got.plt 00000068 0000000000601000 0000000000601000 00001000 2**3  
 CONTENTS, ALLOC, LOAD, DATA  
 22 .data 00000010 0000000000601068 0000000000601068 00001068 2**3  
 CONTENTS, ALLOC, LOAD, DATA  
 23 .bss 00000010 0000000000601078 0000000000601078 00001078 2**3  
 ALLOC  
 24 .comment 00000029 0000000000000000 0000000000000000 00001078 2**0  
 CONTENTS, READONLY
```
Let’s break down the meaning of the new columns:

* **Size**: The size of the section in bytes
* **VMA**: The virtual memory address that the section will be loaded into during program execution
* **LMA**: The load memory address or the memory address that which the section will be loaded in physical memory

Each of the sections listed in the output serves a particular purpose in the program. For instance, the `.bss` section contains uninitiated global variables and `.gnu_hash` is the hash table used for efficient symbol lookup in the dynamic linking process. The full meanings of the different sections can be found [at this link](https://refspecs.linuxbase.org/elf/gabi4+/ch4.sheader.html) towards the bottom of the page.


> A note on terminology: the ELF file used by the linker are called “sections” and the parts used by the loader are called “segments”.

Symbol Table
------------

Finally, the symbol contains information useful for locating and relocating a program’s symbolic definitions and references. It helps in symbol resolution, linking, and debugging.

Symbols are segments of code or information such as functions and variables that can be reused by the program. A symbol table keeps track of the different symbols and their locations.


```
SYMBOL TABLE:  
0000000000400238 l d .interp 0000000000000000 .interp  
0000000000400254 l d .note.ABI-tag 0000000000000000 .note.ABI-tag  
0000000000400274 l d .note.gnu.build-id 0000000000000000 .note.gnu.build-id  
0000000000400298 l d .gnu.hash 0000000000000000 .gnu.hash  
00000000004002c0 l d .dynsym 0000000000000000 .dynsym  
0000000000400410 l d .dynstr 0000000000000000 .dynstr  
00000000004004b8 l d .gnu.version 0000000000000000 .gnu.version  
00000000004004d8 l d .gnu.version\_r 0000000000000000 .gnu.version\_r  
0000000000400518 l d .rela.dyn 0000000000000000 .rela.dyn  
0000000000400560 l d .rela.plt 0000000000000000 .rela.plt  
0000000000400650 l d .init 0000000000000000 .init  
0000000000400670 l d .plt 0000000000000000 .plt  
0000000000400720 l d .text 0000000000000000 .text  
0000000000400af4 l d .fini 0000000000000000 .fini  
0000000000400b00 l d .rodata 0000000000000000 .rodata  
0000000000400b58 l d .eh\_frame\_hdr 0000000000000000 .eh\_frame\_hdr  
0000000000400b98 l d .eh\_frame 0000000000000000 .eh\_frame  
0000000000600e00 l d .init\_array 0000000000000000 .init\_array  
0000000000600e08 l d .fini\_array 0000000000000000 .fini\_array  
0000000000600e10 l d .dynamic 0000000000000000 .dynamic  
0000000000600ff0 l d .got 0000000000000000 .got  
0000000000601000 l d .got.plt 0000000000000000 .got.plt  
0000000000601068 l d .data 0000000000000000 .data  
0000000000601078 l d .bss 0000000000000000 .bss  
0000000000000000 l d .comment 0000000000000000 .comment  
0000000000000000 l df *ABS* 0000000000000000 crtstuff.c  
0000000000400760 l F .text 0000000000000000 deregister\_tm\_clones  
0000000000400790 l F .text 0000000000000000 register\_tm\_clones  
00000000004007d0 l F .text 0000000000000000 \_\_do\_global\_dtors\_aux  
0000000000601080 l O .bss 0000000000000001 completed.7698  
0000000000600e08 l O .fini\_array 0000000000000000 \_\_do\_global\_dtors\_aux\_fini\_array\_entry  
0000000000400800 l F .text 0000000000000000 frame\_dummy  
0000000000600e00 l O .init\_array 0000000000000000 \_\_frame\_dummy\_init\_array\_entry  
0000000000000000 l df *ABS* 0000000000000000 heapedit.c  
0000000000000000 l df *ABS* 0000000000000000 crtstuff.c  
0000000000400c94 l O .eh\_frame 0000000000000000 \_\_FRAME\_END\_\_  
0000000000000000 l df *ABS* 0000000000000000   
0000000000600e08 l .init\_array 0000000000000000 \_\_init\_array\_end  
0000000000600e10 l O .dynamic 0000000000000000 \_DYNAMIC  
0000000000600e00 l .init\_array 0000000000000000 \_\_init\_array\_start  
0000000000400b58 l .eh\_frame\_hdr 0000000000000000 \_\_GNU\_EH\_FRAME\_HDR  
0000000000601000 l O .got.plt 0000000000000000 \_GLOBAL\_OFFSET\_TABLE\_  
0000000000400af0 g F .text 0000000000000002 \_\_libc\_csu\_fini  
0000000000000000 F *UND* 0000000000000000 free@@GLIBC\_2.2.5  
0000000000601078 g O .bss 0000000000000008 stdout@@GLIBC\_2.2.5  
0000000000601068 w .data 0000000000000000 data\_start  
0000000000000000 F *UND* 0000000000000000 puts@@GLIBC\_2.2.5  
0000000000601078 g .data 0000000000000000 \_edata  
0000000000400af4 g F .fini 0000000000000000 \_fini  
0000000000000000 F *UND* 0000000000000000 \_\_stack\_chk\_fail@@GLIBC\_2.4  
0000000000000000 F *UND* 0000000000000000 setbuf@@GLIBC\_2.2.5  
0000000000000000 F *UND* 0000000000000000 printf@@GLIBC\_2.2.5  
0000000000000000 F *UND* 0000000000000000 \_\_libc\_start\_main@@GLIBC\_2.2.5  
0000000000000000 F *UND* 0000000000000000 fgets@@GLIBC\_2.2.5  
0000000000601068 g .data 0000000000000000 \_\_data\_start  
0000000000000000 w *UND* 0000000000000000 \_\_gmon\_start\_\_  
0000000000601070 g O .data 0000000000000000 .hidden \_\_dso\_handle  
0000000000400b00 g O .rodata 0000000000000004 \_IO\_stdin\_used  
0000000000400a80 g F .text 0000000000000065 \_\_libc\_csu\_init  
0000000000000000 F *UND* 0000000000000000 malloc@@GLIBC\_2.2.5  
0000000000601088 g .bss 0000000000000000 \_end  
0000000000400750 g F .text 0000000000000002 .hidden \_dl\_relocate\_static\_pie  
0000000000400720 g F .text 000000000000002b \_start  
0000000000601078 g .bss 0000000000000000 \_\_bss\_start  
0000000000400807 g F .text 0000000000000277 main  
0000000000000000 F *UND* 0000000000000000 fopen@@GLIBC\_2.2.5  
0000000000000000 F *UND* 0000000000000000 \_\_isoc99\_scanf@@GLIBC\_2.7  
0000000000000000 F *UND* 0000000000000000 strcat@@GLIBC\_2.2.5  
0000000000601078 g O .data 0000000000000000 .hidden \_\_TMC\_END\_\_  
0000000000400650 g F .init 0000000000000000 \_init
```
The leftmost column is the memory address of the symbol. The next section represents 7 types of flags that the program can have. For instance, the first row containing the symbol `.interpret` has the flags “l” and “d”. The meaning of these can be found in the `objdump` [man page](https://linux.die.net/man/1/objdump) in the `--syms` section. For instance, the flags on `.interpret` indicate that the symbol is **l**ocal (only visible within the object file) and is a **d**ebugging symbol.

The structure of each entry in the symbol table is defined in the header file `sys/elf.h` or here:


```
typedef struct {  
 Elf32\_Word st\_name;  
 Elf32\_Addr st\_value;  
 Elf32\_Word st\_size;  
 unsigned char st\_info;  
 unsigned char st\_other;  
 Elf32\_Half st\_shndx;  
} Elf32\_Sym;  
  
typedef struct {  
 Elf64\_Word st\_name;  
 unsigned char st\_info;  
 unsigned char st\_other;  
 Elf64\_Half st\_shndx;  
 Elf64\_Addr st\_value;  
 Elf64\_Xword st\_size;  
} Elf64\_Sym;
```
Conclusion
----------

The format of an ELF object file allows for reliable access to important details that aid in program execution such as dynamic linking and storing variables. Tools like `objdump` aid in revealing and understanding these pieces of information and the connections between them.

More Reading
============

* [Oracle Documentation](https://docs.oracle.com/cd/E23824_01/html/819-0690/chapter6-46512.html#scrolltoc) on ELF object file format
* [Objdump Documentation](https://linux.die.net/man/1/objdump)
* [Elf Man Page](https://linux.die.net/man/1/elf)