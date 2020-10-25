# chipguyOS

chipguyOS 

Bare-bones serial-terminal OS for Arduino, for configuring I/O pins and utility "programs".

License: GPLv3

## Features

* A "pin map" for assigning built-in features to specific pins, through the terminal prompt
* A simple EEPROM file system for holding the pin map and other settings
* A specialized wear-leveled EEPROM file type to maintain frequently changing (or incrementing) variables

## Pin map features
* ```pins``` command to list the configured purpose of each pin
* Configured purposes can include connecting them to "programs" linked to the binary (such as infrared receive, NeoPixel output, etc.)
* Configured purposes can include connecting them as "inputs" and "outputs" to programs and defining which states (high, low, floating, pullup-enabled, etc.) are their "active" and "inactive" states
