---
title:  "Lutron Caséta Pico Teardown"
---

![the remote](/assets/images/2025-04-08-lutron-caseta-pico-teardown/1744142366476.png)

I've been messing with Lutron Caséta smart switches around my home recently. I wanted to see what was inside one of my Pico remotes (model number `PJ2-P2B`), so I took it apart.

![label on back of remote](/assets/images/2025-04-08-lutron-caseta-pico-teardown/1744140902994.png)

![internals of the remote](/assets/images/2025-04-08-lutron-caseta-pico-teardown/1744142389762.png)

Inside is a CR2032 coin cell which supposedly lasts up to 10 years, and people on the web agree.

![back of the motherboard](/assets/images/2025-04-08-lutron-caseta-pico-teardown/1744142448173.png)

All the components live on the back of the motherboard, except two metal dome switches for the on/off buttons.

![front of the motherboard](/assets/images/2025-04-08-lutron-caseta-pico-teardown/1744142478903.png)

![zoomed in back of the motherboard](/assets/images/2025-04-08-lutron-caseta-pico-teardown/1744142603605.png)

There's also a loop antenna near the bottom of the backside of the board, though it is difficult to make it out. 

![loop antenna](/assets/images/2025-04-08-lutron-caseta-pico-teardown/1744142680325.png)

The main chip is an Si4010-C2 from Silicon Labs. This is a low-power sub-GHz RF transmitter. [Datasheet.](/assets/docs/Si4010.pdf)

![Si4010-C2 up close](/assets/images/2025-04-08-lutron-caseta-pico-teardown/1744143167825.png)

![Si4010-C2 functional block diagram](/assets/images/2025-04-08-lutron-caseta-pico-teardown/1744142876843.png)

![Si4010-C2 pinout](/assets/images/2025-04-08-lutron-caseta-pico-teardown/1744143036572.png)

From the diagrams, it is clear to me that this chip is very purpose-built for an application like a simple remote control. It's a simple enough circuit that you could easily build your own with a protoboard and a few components. The chip can be purchased [on DigiKey](https://www.digikey.com/en/products/detail/silicon-labs/SI4010-C2-GS/2441251) in small quantities for about 4-5 bucks each. 

![digikey part listing](/assets/images/2025-04-08-lutron-caseta-pico-teardown/1744143398451.png)

Of course if you wanted to make it work with the Lutron system, you'd have to reverse engineer their protocol. I wouldn't be surprised if that info is already on the web.

Anyway, that's a look inside the remote. I'm putting it back together now 🙂 It's going on the wall of my stairwell.