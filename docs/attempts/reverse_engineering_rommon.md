# Reverse-Engineering ROMMON

When I originally ran into the NCC Group's research, I didn't think about making modifications to the firmware image myself. However, I took it as a potential option when I thought *it can't be that hard?* with regards to just finding the execution entrypoint.

While I had read over how good Binwalk was for binary analysis, I didn't dig into it until I ran across this github [write-up](https://gist.github.com/nstarke/ed0aba2c882b8b3078747a567ee00520) on how someone was able to use Ghidra to view the contents of a Cisco IOS image.

In following that write-up loosely to do the same, I was able to utilize Binwalk to discover [three ELF (Executable and Linkable File) formatted-regions](../references/binwalk_analysis/extraction_log.txt) of the firmware and pull them in for analysis. As the rest of the firmware contained a mix of the linux kernel, the rootfs image mentioned before, and either blank space or CRC/hash values, I looked to just those three components for the execution entrypoint.

In reviewing the format of an ELF file, listed [here](../references/executable_documentation/ELF_Format.pdf), I was able to strike two of those files out as utilized by the linux kernel itself (for system calls) and I traced the entrypoint to virtual address 0x100000 (noted by Ghidra). Initially, I even modified a [Hello World 16-bit bootloader](https://medium.com/@g33konaut/writing-an-x86-hello-world-boot-loader-with-assembly-3e4c5bdd96cf) to use that virtual address and tried to load it as an x86 ELF binary. However, it quickly got checked by ROMMON and it was labeled as an invalid image. 

While I knew from a previous attempt that a firmware image with an invalid checksum would be allowed for booting by ROMMON if it was loaded locally with a CF writer (rather than over the network), I anticipated it failing to a degree due to the vast structual differences of my custom bootloader and the firmware image.

Currently, I'm looking to find ways to map the entirety of memory as firmware chips are at much higher addresses (via either adjusting my PowerShell memory pull script or by pulling use hardware tools). Once that's done, I'm looking into utilizing Ghidra and other debugging/analysis tools to trace how ROMMON parses the image for execution.