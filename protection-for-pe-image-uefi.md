<!--- @file
  Protection for PE image UEFI.md 
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

## Protection for PE image

The DXE core may apply a pre-defined policy to set up the NX attribute for the PE data region and the RO attribute for the PE code region.

1.	The image is loaded by the UEFI boot service - `LoadImage()`. If an image is loaded in some other way, the DXE core does not have such knowledge and the DXE core cannot apply any protection.

2.	The image section is page aligned. If an image is not page aligned, the DXE core cannot apply the page level protection.

3.	The protection policy can be based upon a PCD ‘PcdImageProtectionPolicy`. (https://github.com/tianocore/edk2/blob/master/MdeModulePkg/MdeModulePkg.dec) Whenever a new image is loaded, the DxeCore checks the source of the image and then decides the policy of the protection. The policy could be to enable the protection if the sections are aligned, or disable the protection. The platform may choose the policy based upon the need. For example, if a platform thinks the image from the firmware volume should be capable of being protection, it can set protection for IMAGE_FROM_FV. But if a platform is not sure about a PCI option ROM or a file system on disk, it can set no-protection.

There are assumptions for the PE image protection in UEFI:

1.	[Same as SMM] The PE code section and data sections are not merged. If those 2 sections are merged, a #PF exception might be generated because the CPU may try to write a RO data in data section or execute a NX instruction in the code section.

2.	[Same as SMM] The PE image can be protected if it is page aligned. There should not be any self-modifying-code in the code region. If there is, a platform should not set this PE image to be page aligned.

3.	A platform may not disable the XD in the DXE phase. If a platform disables the XD in the DXE phase, the X86 page table will become invalid because the XD bit in page table becomes a RESERVED bit. The consequence is that a #PF exception will be generated. If a platform wants to disable the XD bit, it must happen in the PEI phase.

In EDK II, the DXE core image services calls `ProtectUefiImage()` on image load and `UnprotectUefiImage()` on image unload. (https://github.com/tianocore/edk2/blob/master/MdeModulePkg/Core/Dxe/Image/Image.c) Then `ProtectUefiImageCommon()` (https://github.com/tianocore/edk2/blob/master/MdeModulePkg/Core/Dxe/Misc/MemoryProtection.c) calls `GetUefiImageProtectionPolicy()` to check the image source and protection policy and parses PE alignment. If all checks pass, `SetUefiImageProtectionAttributes()` calls `SetUefiImageMemoryAttributes()`. Finally, `gCpu->SetMemoryAttribute()` sets **EFI_MEMORY_XP** or **EFI_MEMORY_RO** for the new loaded image , or clears the protection for the old unloaded image. When the CPU driver gets the memory attribute setting request, it updates page table.

The X86 CPU driver https://github.com/tianocore/edk2/blob/master/UefiCpuPkg/CpuDxe/CpuDxe.c `CpuSetMemoryAttributes ()` calls  https://github.com/tianocore/edk2/blob/master/UefiCpuPkg/CpuDxe/CpuPageTable.c, `AssignMemoryPageAttributes()` to setup page table.

The ARM CPU driver https://github.com/tianocore/edk2/blob/master/ArmPkg/Drivers/CpuDxe/CpuMmuCommon.c `CpuSetMemoryAttributes()` also has similar capability.

If an image is loaded before CPU_ARCH protocol is ready, the DXE core just skips the setting. Later these images protection will be set in CPU_ARCH callback function – `MemoryProtectionCpuArchProtocolNotify() `(https://github.com/tianocore/edk2/blob/master/MdeModulePkg/Core/Dxe/Misc/MemoryProtection.c).

In `ExitBootServices` event, `MemoryProtectionExitBootServicesCallback() `(https://github.com/tianocore/edk2/blob/master/MdeModulePkg/Core/Dxe/Misc/MemoryProtection.c) is invoked to unprotect the runtime image, because the runtime image code relocation need write code segment at `SetVirtualAddressMap()`.

