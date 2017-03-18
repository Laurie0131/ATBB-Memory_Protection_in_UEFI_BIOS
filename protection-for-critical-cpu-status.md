<!--- @file

  Protection for Critical CPU status .md for 
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
## Protection for critical CPU status

Besides the PE image, the Intel X86 architecture has some special architecture-specific regions that need to be protected as well.

### SMM EntryPoint

When a hardware SMI occurs, the Intel X86 CPU jumps to an SMM entry point in order to execute the code at this location. This SMM entry point is not inside of a normal PE image, so we also need to protect this region. See the bottom right of figure 2.

According to [[IA32SDM][1]], the SMM entry point is at a fixed offset from SMBASE. In EDK II, the SMBASE and the SMM save state area are allocated at https://github.com/tianocore/edk2/blob/master/UefiCpuPkg/PiSmmCpuDxeSmm/PiSmmCpuDxeSmm.c `PiCpuSmmEntry()`. Both the SMM entry point and the SMM save state are allocated as a CODE page. Later https://github.com/tianocore/edk2/blob/master/UefiCpuPkg/PiSmmCpuDxeSmm/SmmCpuMemoryManagement.c `PatchSmmSaveStateMap()` patches the SMM entry point to be read-only and the SMM save state to be non-executable.

### GDT/IDT

The GDT defines the base address and the limit of a code segment or a data segment. If the GDT is updated, the code might be redirected to a malicious region. As such, the GDT should be set to read-only.

The IDT defines the entry point of the exception handler. If the IDT is updated, the malicious code may trigger an exception and jump to a malicious region. As such, the IDT should be set to read-only as well.

This work is done by `PatchGdtIdtMap()` at https://github.com/tianocore/edk2/blob/master/UefiCpuPkg/PiSmmCpuDxeSmm/X64/SmmFuncsArch.c.

However, the IA32 version GDT cannot be set to read-only if the stack guard feature is enabled. (https://github.com/tianocore/edk2/blob/master/UefiCpuPkg/PiSmmCpuDxeSmm/Ia32/SmmFuncsArch.c) The reason is that the IA32 stack guard needs to use a "_task switch_" to switch the stack, and the task switch needs to write the GDT and Task-State Segment (TSS). The X64 version of the GDT does not have such a problem because the X64 stack guard uses “_interrupt stack table (IST)_” to switch the stack. For details of the stack switch and exceptions, please refer to [[IA32SDM][1]].

### Page Table

In an X86 CPU, we rely on the page table to set up the read-only or non-executable region. In order to prevent the page table itself from being updated, we may need to set the page table itself to be read-only.

The work is done at https://github.com/tianocore/edk2/blob/master/UefiCpuPkg/PiSmmCpuDxeSmm/X64/PageTbl.c `SetPageTableAttributes()`.

However, setting a page table to be read-only may break the original dynamic paging feature in SMM. There is a (PCD) ```PcdCpuSmmStaticPageTable ``` to determine if the platform wants to enable the static page table or the dynamic page table.

If ```PcdCpuSmmStaticPageTable``` is FALSE, the PiSmmCpu uses the original dynamic paging policy, namely the the PiSmmCpu only sets 4GiB paging by default. If the PiSmmCpu needs to access above 4GiB memory locations, a page fault exception (#PF) exception is triggered and an above-4GiB mapping is created in the page fault handler.

If ```PcdCpuSmmStaticPageTable``` is TRUE, the PiSmmCpu will try to set the read-only attribute for the page table.

Figure 2 shows the mapping of the protection.

![](/assets/Fig2 - Mapping of Protection in SMM.jpg)

Figure 2 Mapping of Protection in SMM



