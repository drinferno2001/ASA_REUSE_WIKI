# Purpose And Initial Premise

The purpose of this project is to see whether I could run Mikrotik's Router OS (or any custom code) on a locked-down ASA 5505. 

I originally had bought/acquired a whole bunch of Cisco equipment on EBay for CCNA studies and was going to re-sell it when I came across this article on [Medium](https://medium.com/@DomPolizzi/install-opnsense-and-linux-on-cisco-asa-59995dd6d60f).

As the article showed a pretty easy way of getting into the bootloader via VGA pin headers, I wondered if I could do the same with an ASA 5505 I had on hand. As I'm currently in the midst of upgrading my homelab with Mikrotik switches and came to be interested in RouterOS' capabilities, I wondered if I could re-purpose the device as a small travel firewall.

However, when opening up the device and doing some research online, I got to learn that the older ASA models (5505 included), don't expose VGA pins and I wanted to see if I could bypass it.

This repository documents my efforts and any relevant resources that I come across.