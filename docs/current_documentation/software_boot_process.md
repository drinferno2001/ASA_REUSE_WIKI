# Software (Device Boot Process)

In order to understand or get an idea of what was needed to start cracking the 5505, I needed to review the boot process. I've got it simplified below:

**EMBEDDED BIOS -> ROMMON -> CISCO ASA**

While the Cisco ASA software itself drives the firewall, it isn't the only piece in the boot process. Cisco ASA is itself an OS in the form of a .bin (binary) file that's loaded by an on-chip firmware called **ROMMON**. This chip is responsible for both booting the OS and providing recovery in case the ASA OS is un-authentic or corrupted. The downside with this firmware though is that it is only designed to boot Cisco-provided firmware images and we can't overwrite it as a bootloader.

While delving into how I could find a way around ROMMON, I was able to track down some documentation regarding the BIOS in use. Specifically, it looks to be a piece of embedded software called, quite-literally, **EMBEDDED SOFTWARE** and it was produced by a company (General Software, Inc.) that has since been bought out. 

While the version on-board the device is much older [1.0(12)13] than the 4.1 or 4.3 documentation I found, it still pretty essential in that I'm not able to find anything else with regards to the BIOS software (or the included debugger).