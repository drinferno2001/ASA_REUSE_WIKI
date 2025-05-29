# Board Reference

![Screenshot](./board_images/board_layout.png)

## Component Reference

1. Companion processor: AMD Geode CS35536 (Southbridge)
    - Communicates with primary CPU and compact flash
    - Communicates with USB
    - Handles IO
    - Has RTC (Real-Time Clock) - Powered by CMOS battery

2. Main Processor: AMD Geode XL600, x86 CPU running at 500MHz

3. JTAG?

4. Physical IO chip for Layer1: Marvel 88ACS06 (octal PHY)
    - 8 IO Ports To 8 100 MB Ethernet Ports

5. ROMMON: SST 49LF016C 2MB Flash chip

6. ASA OS: CF (Compact Flash) Card

7. Onboard accelerator/ASIC: 
    - Front (FPGA) - Handles VPN
    - Back (Cavium Nitrox Lite security macro processor) - Handles Encryption

8. NVRAM: ST Microelectronics 24CD4WP (4Kbit EEPROM)

9. Security microcontroller for Flash: Atmel 12836RCT
    - Prevents cleartext data at rest in flash

10. PoE controller: Linear Technology LTC4259ACGW

11. DDR RAM Module

12. Serial Console: ADM3202 RS232 transceiver