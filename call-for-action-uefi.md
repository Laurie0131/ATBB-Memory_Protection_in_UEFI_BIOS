<!--- @file
  Call for Action UEFI.md 
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
## Call for action

In order to support UEFI memory protection, the firmware need configure UEFI driver to be page aligned:

1.	Override link flags below to support UEFI runtime attribute table, so that OS can protect the runtime memory.
```css 
[BuildOptions.IA32.EDKII.DXE_RUNTIME_DRIVER,  
 BuildOptions.X64.EDKII.DXE_RUNTIME_DRIVER]
 MSFT:*_*_*_DLINK_FLAGS = /ALIGN:4096 
 GCC:*_*_*_DLINK_FLAGS = -z common-page-size=0x1000
```
2.	Override link flags below to support UEFI memory protection.
 ```css
 [BuildOptions.common.EDKII.DXE_DRIVER, 

 BuildOptions.common.EDKII.DXE_CORE, 
 BuildOptions.common.EDKII.UEFI_DRIVER,
 BuildOptions.common.EDKII.UEFI_APPLICATION]
 MSFT:*_*_*_DLINK_FLAGS = /ALIGN:4096 
 GCC:*_*_*_DLINK_FLAGS = -z common-page-size=0x1000
 ```

3.	Evaluate if the UEFI memory size is big enough to hold the split page table.
4.	Evaluate if the DXE image can be protected.
5.	Set proper `gEfiMdeModulePkgTokenSpaceGuid.PcdImageProtectionPolicy`.
6.	Set proper `gEfiMdeModulePkgTokenSpaceGuid.PcdDxeNxMemoryProtectionPolicy`.

#### Summary

This section introduces the memory protection in UEFI.

[1]: https://github.com/tianocore-docs/Docs/raw/master/White_Papers/A_Tour_Beyond_BIOS_Memory_Map_And_Practices_in_UEFI_BIOS_V2.pdf "MemMap"

[2]: http://uefi.org "UEFI"

[3]: https://github.com/tianocore-docs/Docs/raw/master/White_Papers/A_Tour_Beyond_BIOS_Securiy_Enhancement_to_Mitigate_Buffer_Overflow_in_UEFI.pdf "Security Enhancment"