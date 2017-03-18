<!--- @file

  Memory Protection in SMM.md for 
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

#Memory Protection in SMM

The SMM is an isolated execution environment according to Intel® 64 and IA-32 Architectures Software Developer’s Manual [[IA32SDM][1]]. The UEFI Platform Initialization [[PI][2]] specification volume 4 defines the SMM infrastructure. Figure 1 shows the SMM memory protection. **RO** designates read-only memory. **XD **designates execution-disabled memory.

![](/assets/Fig1- SMRAM memory protection.jpg)
Figure 1 - SMRAM memory protection

[1]: https://software.intel.com/en-us/articles/intel-sdm "IA32SDM"
[2]: http://uefi.org "PI Spec"



[3]: https://msdn.microsoft.com/en-us/library/windows/hardware/dn495660(v=vs.85).aspx#wsmt "WindowsWSMT"

[4]: http://download.microsoft.com/download/1/8/A/18A21244-EB67-4538-BAA2-1A54E0E490B6/WSMT.docx "WindowsWSMT docx"
[5]: https://msdn.microsoft.com/en-us/library/windows/hardware/dn614617 "MicrosoftHV"
[6]: https://github.com/tianocore-docs/Docs/raw/master/White_Papers/A_Tour_Beyond_BIOS_Secure_SMM_Communication.pdf "SecureSmmComm"


[7]: https://en.wikipedia.org/wiki/Address_space_layout_randomization "ASLR"

[8]: https://en.wikipedia.org/wiki/Return-oriented_programming "ROP"



 
