<
!--- @file

Protection for PE image.md for
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
EVENT SHALL TIANOCORE PROJECT BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS;
OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR
OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS DOCUMENTATION, EVEN IF
ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

-->




## Protection for PE image
In UEFI/PI firmware, the SMM image is a normal PE/COFF image loaded by the SmmCore. If a given section of the SMM image is page aligned, it may be protected according to the section attributes, such as read-only for the code and non-executable for data. See the top right of figure 1.

In EDK II, the PiSmmCore (https://github.com/tianocore/edk2/blob/master/MdeModulePkg/Core/PiSmmCore/MemoryAttributesTable.c) checks the PE image alignment and builds an `EDKII_PI_SMM_MEMORY_ATTRIBUTES_TABLE ` (https://github.com/tianocore/edk2/blob/master/MdeModulePkg/Include/Guid/PiSmmMemoryAttributesTable.h) to record such information. If the PI SMM image is not page aligned, this table will not be published. If the `EDKII_PI_SMM_MEMORY_ATTRIBUTES_TABLE` is published, that means the `EfiRuntimeServicesCode` contains only code and it is ``EFI_MEMORY_RO``, and the `EfiRuntimeServicesData` contains only data and it is `EFI_MEMORY_XP`.

Later the PiSmmCpu driver (https://github.com/tianocore/edk2/blob/master/UefiCpuPkg/PiSmmCpuDxeSmm/SmmCpuMemoryManagement.c)` SetMemMapAttributes()` API consumes the ``EDKII_PI_SMM_MEMORY_ATTRIBUTES_TABLE`` and sets the page table attribute.

There are several assumptions to support the PE image protection in SMM:

1.	The PE code section and data sections are not merged. If those 2 sections are merged, a #PF exception might be generated because the CPU might try to write a RO data item in the data section or execute a non-executable (NX) instruction in code section.
2.	The PE image can be protected if it is page aligned. There should not be any self-modified-code in the code region. If there is, a platform should not set this PE image to be page aligned.

A platform may disable the XD in the UEFI environment, but this does not impact the SMM environment. The SMM environment may choose to always enable the XD upon SMM entry, and restore the XD state at the SMM exit point.

