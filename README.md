# chipguyOS by chipguyhere

chipguyOS is a bare-bones terminal OS for Arduino that allows an installer to configure I/O pins and utility "programs".  In this context, an "installer" is someone who is unable or uninterested in compiling or sending a sketch to an Arduino board, but is able or willing to connect and configure it through a generic serial terminal program.   chipguyOS is provided in the form of a library to be added to an Arduino sketch, and has been tested on Arduino Uno, Nano, and Mega.

The example sketch, as provided, is far more useful than simply a demonstration, as it exposes the library's terminal prompt to the Arduino serial port, where the library's communication and dispatch features can be provisioned to make a device that's able to send events from "point A" to "point B".

Your custom sketch can add additional functionality.  Your sketch can allow or regulate access to the serial configuration prompt simply by deciding whether and when to continue to pass serial I/O to the serial terminal methods in the library.

This library is licensed under the GNU Public License, version 3 (see the included LICENSE file for details).

## Features

* A "pin map" for assigning built-in features to specific pins, through the terminal prompt
* A simplified EEPROM file system for holding configuration settings, the pin map, and other persistent variables for your sketch (including wear leveling for counters and things that change frequently)
* A simplified event dispatch system
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
