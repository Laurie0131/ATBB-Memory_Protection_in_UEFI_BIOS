<!--- @file

  Limitation.md for 
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

# Memory Protection in SMM

## Limitation

Setting up RO and NX attribute for SMRAM is a good enhancement to prevent a code overriding attack. However it has some limitations:

1. It cannot resist a Return-Oriented-Programming \(ROP\) attack. \[[ROP](https://en.wikipedia.org/wiki/Return-oriented_programming "ROP")\]. We might need ASLR to mitigate the ROP attack. \[[ASLR](https://en.wikipedia.org/wiki/Address_space_layout_randomization "ASLR")\] With the code region randomized, an attacker cannot accurately predict the location of instructions in order to leverage gadgets.

2. Not all important data structure are set to Read-Only. This is the current SMM driver limitation. The SMM driver can be updated to allocate the important structures to be read-only instead of a read-write global variable.

To set not-present bit for non-fixed DRAM region in SmmReadyToLock is a good enhancement to enforce the protection policy. However, it cannot cover below cases:

1. Memory Hot Plug. Take a server platform as the example, A RAS server may hot plug more DRAM during OS runtime, and rely on SMM to initialize those DRAM. This SMM Memory Initialization module may need access the DRAM for the memory test.

2. Memory Mapped IO \(MMIO\). Ideally, not all MMIO regions are configured to be accessible to SMM. Some MMIO BARs are important such as VTd or SPI controller.  VTd BAR is important because OS need setup VTd to configuration the DMA protection. SPI controller BAR is important because BIOS SMM handler need access it to program the flash device. It should be a platform policy to configure which one should be accessible. The SMI handler must consider the case that the MMIO BAR might be modified by the malicious software and check if the MMIO BAR is in the valid region.

