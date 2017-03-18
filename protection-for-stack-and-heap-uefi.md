<!--- @file
  Protection for stack and heap UEFI.md 
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
## Protection for stack and heap

The [[UEFI][2]] specification allows 

>"Stack may be marked as non-executable in identity mapped page tables." 

As such, we set up the NX stack (https://github.com/tianocore/edk2/blob/master/MdeModulePkg/Core/DxeIplPeim/X64/VirtualMemory.c, `CreateIdentityMappingPageTables()`).

The heap protection is based upon the policy, because we already observed some unexpected usage in [[MemMap][1]] white paper. A platform needs to  configure a PCD `PcdDxeNxMemoryProtectionPolicy` 

(https://github.com/tianocore/edk2/blob/master/MdeModulePkg/MdeModulePkg.dec) to indicate which type of memory can be set to NX in the page table. The DxeCore `ApplyMemoryProtectionPolicy()` (https://github.com/tianocore/edk2/blob/master/MdeModulePkg/Core/Dxe/Misc/MemoryProtection.c) consumes the PCD after the memory allocation service and sets NX attribute for the allocated memory by using CPU_ARCH protocol.

Before CPU_ARCH protocol is ready, the protection takes no effect. In CPU_ARCH callback function – `MemoryProtectionCpuArchProtocolNotify() `(https://github.com/tianocore/edk2/blob/master/MdeModulePkg/Core/Dxe/Misc/MemoryProtection.c), the `InitializeDxeNxMemoryProtectionPolicy()` is called to get current memory map and setup the NX protection.

In addition, we may use some special techniques, such as the guard page, to apply the protection for the allocated memory in order to detect a buffer overflow. This is discussed in [[SecurityEnhancement][3]] white paper.


[2]: http://uefi.org "UEFI"
[3]: https://github.com/tianocore-docs/Docs/raw/master/White_Papers/A_Tour_Beyond_BIOS_Securiy_Enhancement_to_Mitigate_Buffer_Overflow_in_UEFI.pdf "Security Enhancment"
