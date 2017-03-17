<!--- @file

  Non SMRAM access in SMM.md for 
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

## Non SMRAM access in SMM

Besides the SMRAM, the SMM memory protection also limits the access to the non-SMRAM region.

First, the non-SMRAM region must be set to be non-executable because the SMM entities should not call any code outside SMRAM. Code outside of SMRAM might be controlled by malicious software.

This protection work is done by `InitPaging()` at [https:\/\/github.com\/tianocore\/edk2\/blob\/master\/UefiCpuPkg\/PiSmmCpuDxeSmm\/SmmProfile.c](https://github.com/tianocore/edk2/blob/master/UefiCpuPkg/PiSmmCpuDxeSmm/SmmProfile.c)

Second, because of the security concerns regarding SMM entities accessing VMM memory, [[WindowsWSMT][3]], [[Wsmt.docx][4]] and [[MicrosoftHV][5]] introduced the Windows SMM Security Mitigations Table \(WSMT\). A platform needs to report the WSMT table in order to declare that the SMI handler will validate the SMM communication buffer.

[3]: https://msdn.microsoft.com/en-us/library/windows/hardware/dn495660(v=vs.85).aspx#wsmt "WindowsWSMT"

[4]: http://download.microsoft.com/download/1/8/A/18A21244-EB67-4538-BAA2-1A54E0E490B6/WSMT.docx "WindowsWSMT docx"

[5]: https://msdn.microsoft.com/en-us/library/windows/hardware/dn614617 "MicrosoftHV"


As we discussed in \[[SecureSmmComm](https://github.com/tianocore-docs/Docs/raw/master/White_Papers/A_Tour_Beyond_BIOS_Secure_SMM_Communication.pdf "SecureSmmComm")\], the SMI handler should check if the SMM communication buffer is from a fixed region, \(EfiReservedMemoryType\/ EfiACPIMemoryNVS\/ EfiRuntimeServicesData\/ EfiRuntimeServicesCode\). However, this is a passive check. If a SMI handler does not include such a check, it is hard to detect.

A better way is to use an active check. The PiSmmCpu driver sets the non-fixed DRAM region \(EfiLoaderCode\/ EfiLoaderData\/ EfiBootServicesCode\/ EfiBootServicesData\/ EfiConventionalMemory\/ EfiUnusableMemory\/ EfiACPIReclaimMemory\) to be not-present in the page tables after the SmmReadyToLock event.

As such, if a platform SMI handler does not include the check recommended in \[[SecureSmmComm](https://github.com/tianocore-docs/Docs/raw/master/White_Papers/A_Tour_Beyond_BIOS_Secure_SMM_Communication.pdf "SecureSmmComm")\], the system will get \#PF exception within SMM on such an attack.

This protection work is done by `SetUefiMemMapAttributes()` at [https:\/\/github.com\/tianocore\/edk2\/blob\/master\/UefiCpuPkg\/PiSmmCpuDxeSmm\/SmmCpuMemoryManagement.c](https://github.com/tianocore/edk2/blob/master/UefiCpuPkg/PiSmmCpuDxeSmm/SmmCpuMemoryManagement.c).

Figure 3 shows final image layout.

![](/assets/Fig3 - Page table enforced memory layout.jpg)

Figure 3 Page table enforced memory layout

The assumption for non-SMRAM access in SMM is described in \[[SecureSmmComm](https://github.com/tianocore-docs/Docs/raw/master/White_Papers/A_Tour_Beyond_BIOS_Secure_SMM_Communication.pdf "SecureSmmComm")\].

Besides that, this solution assumes that all DRAM regions are added to the Global Coherency Domain \(GCD\) management before EndOfDxe, so that the UEFI memory map can return all DRAM regions. If there are more regions added to the GCD after EndOfDxe, those regions are not set to not-present in the page table.

NOTE: The SMM does not set the not-present bit for the GCD **EfiGcdMemoryTypeNonExistent** memory, because this type of memory may be converted to the other types, such as **EfiGcdMemoryTypeReserved**, or **EfiGcdMemoryTypeMemoryMappedIo**, which might be accessed by the SMM later.
