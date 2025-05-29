# Potential Weakspots/Previous Attempts

### BOARD INTERFACES

#### DISCOVERED JTAG INTERFACE?

While disassembling the device, I came across a series of pins (curiously hidden under a barcode) that I thought was originally a place where I could solder VGA pins. (At the time, I was thinking this model just didn't have them like the others did). 

However, upon further research, I determined that it was actually a [**JTAG**](./board.md) interface (see #3), an interface standard that came out in the late 80s/early 90s to simplfy board testing. While used for board testing, the interface also had the benefit in it provided access to all of the signals on all traces on the device's motherboard.

#### UNIDENTIFIED INTERFACES (POTENTIAL UART?)

At the same time, following the quick hardware hacking introduction included [here](/hardware_hacking_references/Hardware.Hacking.Methodology-Jeremy.Brun-v1.0.pdf), I also did various voltage tests against a manner of many other pins located around the JTAG interface and main CPU (in an attempt to look for a UART interface). However, I wasn't able to find any voltage that would've indicated transmit, receive, or Vcc pins. Even on bootup, none of the other interfaces produced a fluctuating voltage that would've indicated the movement of data (logging specifically). See [here](/board_layout/board_pins.txt) for a rough notepad logging of results.

I also tested the JTAG interface and tried to match it using an, admittedly, older guide found [here](http://www.jtagtest.com/pinouts/) but I wasn't able to find a matching format. It was no surprise that I couldn't match it up as this was Cisco and, as I learned, x86 JTAG interfaces were harder to access. I would've potentially needed an expensive and specialized device but I didn't research that far into it for reasons listed below.

#### PUSHING IT ASIDE

While I had looked into the possibility of utilizing hardware hacking techniques to access the device (even just to pull firmware at least), I've stayed away from it roughly due to not wanting to break or damage the hardware. I don't have a lot of experience when it comes to hardware hacking and I've had a tendency in the past to break hardware pretty easily doing just doing repairs on stuff. 

While I could find another ASA 5505 to purpose, I'd rather still have the original device as finding a purpose for that specific device was the goal of the project. This was just due to me having it on-hand at the time.

With that in mind, this became a last resort. At the same time, I read online that a lot of JTAG interfaces are disabled and I have a feeling that Cisco's done this as well given how locked down everything else of their's is. 

However, curiously, they did have a barcode placed over the pinholes and ... it doesn't make sense to cover up something that doesn't allow access to anything....

### SUBVERTING THE BOOT PROCESS

While looking online initially to see if others had done something similar, I came across the following articles: 

- [Rapid7](https://www.rapid7.com/blog/post/2016/06/14/asa-hack/)
- [NCC Group](https://www.nccgroup.com/us/research-blog/cisco-asa-series-part-one-intro-to-the-cisco-asa/)

#### MODIFYING ASA FIRMWARE IMAGES FOR ROOT SHELL

The first article caught my eye as it showcased how easy it would be to modify an ASA OS image to boot into a root shell. All you had to do was load the firmware image in a hex editor, update some linux kernel parameters (it used linux?), load the firmware on the device, and power it on. In doing so, you were able to access a root shell that provided access to the system before the Lina binary was able to take over.

However, I wasn't able to find much use here as I was only gaining access to the rootfs image (essentially looking at the base for a ramdisk). Busybox provided a lot of utilities for use there (and Lina was accessible) but I couldn't really find much else there. 

#### DISCOVERING A DEBUG SCREEN

In finding that article, I was able to find the second article and pull up the NCC Group and research they had done into the ASA Lina binary close to 10 years before. While most of it is irrelevant from the point of view of this project (they to modify the firmware image to introduce debugging facilities for Lina and I was looking to subvert the boot process entirely), I did come across a very minor comment (in the **Convenience Tips** section) that opened another avenue:

> While replacing a CF card that was malfunctioning, we noticed that the ASA 5505 (and likely other models as well) appears to have an 8GB size limit for CF cards. Inserting a 16GB CF card will simply cause the BIOS to fail POST. This size limit doesn’t appear to be well documented and we didn’t investigate further. Interestingly this drops you to an extended BIOS debug (EBDEBUG) shell (which doesn’t seem to be well documented).

In wanting to try and replicate this, I purchased a 16 GB CF card, inserted it into the ASA, and was quickly able to force the system to boot to a BIOS-level debugger. Evidently, the larger card would cause some bootup code to throw a divide-by-zero error and the firmware was programmed to drop into a debug shell for handling. A quick Google search of the prompt for the shell (called **EBDEBUG**) led me to the [BIOS documentation](bios_documentation/) that's listed in this repo.

With reading into the documentation and utilizing [OSDev.com](https://wiki.osdev.org/Expanded_Main_Page) to get an better understanding of real-mode, interrupts, and the low-memory model, I realized that the debugger provided me with everything I could use for potentially kicking off custom code. This included the ability to execute instructions directly, moving across memory by adjusting the EIP register, and even with reading/writing to flash storage.

As of currently, I'm still looking into utilizing the BIOS debugger for loading and I've even followed a [tutorial](https://medium.com/@g33konaut/writing-an-x86-hello-world-boot-loader-with-assembly-3e4c5bdd96cf) to build a 16-bit Hello World bootloader to test it. It looks like the venture might be possible but it wouldn't be entirely feasible as the code would have to be re-executed on every bootup. This led me to another option that I'm looking at concurrently (listed below).

#### MODIFYING ROMMON

When I originally ran into the NCC Group's research, I didn't think about making modifications to the firmware image myself. However, I took it as a potential option when I thought 'it can't be that hard?' with regards to just finding the execution entrypoint.

While I had read over how good Binwalk was for binary analysis, I didn't dig into it until I ran across this github [write-up](https://gist.github.com/nstarke/ed0aba2c882b8b3078747a567ee00520) on how someone was able to use Ghidra to view the contents of a Cisco IOS image.

In following that write-up loosely to do the same, I was able to utilize Binwalk to discover [three ELF (Executable and Linkable File) formatted-regions](/binwalk_analysis/extraction_log.txt) of the firmware and pull them in for analysis. As the rest of the firmware contained a mix of the linux kernel, the rootfs image mentioned before, and either blank space or CRC/hash values, I looked to just those three files for the execution entrypoint.

In reviewing the format of an ELF file, listed [here](/executable_documentation/ELF_Format.pdf), I believe I was able to strike two of those files out as utilized by the linux kernel itself for system calls and I traced the entrypoint to virtual address 0x100000 (noted by Ghidra). Initially, I even modified the Hello World 16-bit bootloader to use that virtual address and tried to load it as an x86 ELF binary. However, it quickly got checked by ROMMON and it was labeled as an invalid image. 

While I knew from a previous attempt that a firmware image with an invalid checksum would be allowed for booting by ROMMON if it was loaded locally with a CF writer (rather than over the network), I anticipated it failing to a degree due to the vast structual differences of my bootloader and the firmware image (it's one thing for the image to be considered valid and it's another for it to be so different that it's not even recognizable by ROMMON).

I also didn't realize at the time that ROMMON was running in protected mode (which should've been obvious with the first virtual address being the first address past the low memory region (> 1 MB)) and that I would have to modify my bootloader to run in 32-bit mode and to run as a standard binary (rather than an MBR-executable).

Currently, that's where I'm at. I'm looking into the specifics of protected mode (so that I can adjust the binary) and I'm looking into the possibility of stripping an existing Cisco firmware image to the bare minimum to find what needs to be recognized by ROMMON.