<!--- @file
  Life cycle of the protection UEFI.md 
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

## Life cycle of the protection

The UEFI image protection starts when the CpuArch protocol is ready. The UEFI runtime image protection is torn down at `ExitBootServices()`, the runtime image code relocation need write code segment at `SetVirtualAddressMap()`. We cannot assume OS/Loader has taken over page table at that time.

The UEFI heap protection also starts when the `CpuArch` protocol is ready.

The UEFI stack protection starts in `DxeIpl`, because the region is fixed and it can set directly.

The UEFI firmware does not own page tables after `ExitBootServices()`, so the OS would have to relax protection of runtime code pages across `SetVirtualAddressMap()`, or delay setting protections on runtime code pages until after `SetVirtualAddressMap()`. OS may set protection on runtime memory based upon EFI_MEMORY_ATTRIBUTES_TABLE later.

