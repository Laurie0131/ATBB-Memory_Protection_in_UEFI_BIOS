<!--- @file
  Size Overhead UEFI.md 
  for A Tour Beyond BIOS - Memory Protection in UEFI BIOS
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
## Size Overhead

1.	**Runtime memory overhead (visible to OS)**
:	The size overhead of the runtime PE image is the same as the overhead of the SMM PE image.  If a platform has n runtime images, the average amount overhead is `6K * n`.

2.	**Boot time memory overhead (invisible to OS)**
:	The size of the overhead for the boot time PE image is the same as the overhead of the SMM PE image. If a platform has n boot time images, the average overhead is `6K * n`.

If the NX protection for data is enabled, the size of the page table is increased because we need set fine granularity page level protection.

The size overhead of the boot time page table is also same as for the SMM static page table. Please refer to the SMM section for the size calculation based upon the 1G paging capability and max supported address bit.

