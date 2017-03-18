<!--- @file

  SMRAM Size Overhead.md for 
    A Tour Beyond BIOS - Memory Protection in UEFI BIOS
  Copyright (c) 2017, Intel Corporation. All rights reserved.<BR>
  Redistribution and use in source (original document form) and 'compiled'
  forms (converted to PDF, epub, HTML and other formats) with or without
   modification, are permitted provided that the following conditions are met:
  1) Redistributions of source code (original document form) must retain the
     above copyright notice, this list of conditions and the following
     disclaimer as the first lines of this file unmodified.
  2) Redistributions in compiled form (transformed to other DTDs, converted to
     PDF, epub, HTML and other formats) must reproduce the above copyright
     notice, this list of conditions and the following disclaimer in the
     documentation and/or other materials provided with the distribution.
  THIS DOCUMENTATION IS PROVIDED BY TIANOCORE PROJECT "AS IS" AND ANY EXPRESS OR
  IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF
  MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO
  EVENT SHALL TIANOCORE PROJECT  BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
  SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
  PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS;
  OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
  WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR
  OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS DOCUMENTATION, EVEN IF
   ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

-->




## SMRAM Size Overhead

### PE image

In order to protect the PE code and data sections, we must set the PE image section alignment to be 4K.

In EDK II, the default PE image alignment is 0x20 bytes. Assuming one PE image has 3 sections \(1 header, 1 code section, 1 data section\), average overhead for one PE image is `(4K * 3) / 2 = 6K`.

If a platform has n SMM images, the average of the overhead is `6K * n`.

### Page Table

In order to protect the page table itself, we must use the static page table instead of the dynamic on-demand page table.

The size of the dynamic paging is fixed. We need 6 fixed pages \(24K\) and 8 on-demand pages \(32K\). The total size of the page table is 56K in this case.

The size of the static page table depends upon 2 things: 1\) 1G paging capability, 2\) max supported address bit. A rough estimation is below:

1.    If 1G paging is supported,
* 32 bit addressing need \(1+1+4\) pages = 24K. \(still use 2M paging for below 4G memory\)
* 39 bit addressing need \(1+1+4\) pages = 24K.
* 48 bit addressing need \(1+512\) pages = 2M.
* If 1G paging is not supported, 2M paging is used.
* 32 bit addressing need \(1+1+4\) pages = 24K.
* 39 bit addressing need \(1+1+512\) pages = 2M.
* 48 bit addressing need \(1+512+512_512\) pages = 1G. &lt; - This seems __**_\*not_**_ acceptable.

The maximum address bit is determined by the \(CPU\_HOB\) if it is present, or the physical address bit returned by the CPUID instruction if the CPU\_HOB is not present. \([https:\/\/github.com\/tianocore\/edk2\/blob\/master\/UefiCpuPkg\/PiSmmCpuDxeSmm\/X64\/PageTbl.c](https://github.com/tianocore/edk2/blob/master/UefiCpuPkg/PiSmmCpuDxeSmm/X64/PageTbl.c), `CalculateMaximumSupportAddress()`\) A platform may set the CPU\_HOB based upon the addressing capability of the memory controller or the CPU.

