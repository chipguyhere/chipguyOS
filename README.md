# chipguyOS

chipguyOS 

Bare-bones serial-terminal OS for Arduino, for configuring I/O pins and utility "programs", in the form of a library.  The library provides its services to your sketch, which allows (or regulates) access to the serial configuration prompt by calling (or not calling) ```serialTerminalMonitor()``` from within your ```loop()```.

License: GPLv3

## Features

* A "pin map" for assigning built-in features to specific pins, through the terminal prompt
* A simplified EEPROM file system for holding configuration settings, the pin map, and other persistent variables for your sketch (including wear leveling for counters and things that change frequently)
* A simplified networking system based around 32-byte messages, sendable over nRF24L01+ packet radio, infrared, I2C, SPI, serial, or Ethernet

## Pin map features
* ```pins``` command to list the configured purpose of each pin
* ```pin``` command to set a pin (e.g. _pin 6=H_ set pin 6 as an output high on boot)
* Pins can connect to your sketch or other "programs" linked to the binary (such as infrared receive, piezo beeper, etc.)
* Pins can connect to programs as "inputs" and "outputs" to programs with custom definition of their "active" and "inactive" states (e.g. high, low, floating, pullup-enabled, etc.)

## Networking features
* A "transmit" program that sends events somewhere else when they happen.
* A "receive" program that can receive those events from some other device.  (A simple example is an input on one device can trigger an output on another device)
* AES encryption and counters on packets.  Packets are fixed at 32 bytes.
* Acknowledgment mode (packet is regenerated and retransmitted at lengthening intervals until acknowledged).
* Non-acknowledgment / broadcast mode (packet is sent with no acknowledgment requested or expected, with retries for good measure)
