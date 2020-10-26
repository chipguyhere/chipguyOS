EEPROM file format by chipguyhere
License: GPLv3

## Header

The first byte (0) of the EEPROM is not set.  It's a "don't care" value.  It's understood that this is the most likely byte to see corruption.

Bytes 1-7 of EEPROM are the characters "chipguy".  Only five of them need to be present and correct for the header to be considered valid.

The first record starts at byte 8.  If there are no records, the bytes at 8 should be 0xFF 0xFF 0xFF.

## Records

Records have the following:
* 3 bytes to indicate length and existence
* 2 bytes to indicate the "filename"
* n bytes for the record content
* 2 bytes for optional CRC (bytes of 0xFF mean CRC not present)

The first three bytes of a record indicate the length and existence of a record.  For normal records, these three bytes
are three copies of the length value.  The length includes all bytes including the CRC.

Only two need to be correct.  When any two bytes are the same and not 0xFF, that is the record length.  If any two bytes are 0xFF, the record is
deleted free space, in the length of the remaining byte (which shall not be 0xFF).  If all three bytes are different or they are all 0xFF, there
is no valid record.  The next valid record, if any, could be found by scanning for 3 identical bytes that are followed by a correct CRC.

The CRC bytes are optional and are used for determining which is the "best" copy of a record if more than one copy is present.
The bytes represent a CRC-16 over the entire record (including with the three length bytes).  The value 0xFF indicates that the
CRC byte is not present.  However, since the value 0xFF also appears in CRC's, the value 0x7F shall compare as equal when the
actual CRC byte is 0x7F.

If a record is deemed "important" (all configuration records that only exist once are "important") there should be two copies of the record.
The first record with a correct CRC is the record that should be used.  If unavailable, a record with no CRC is preferred over one with an
incorrect CRC.

### How a record is updated

* Find the two copies of the record (copy1 and copy2).  Create them if they don't exist.
* If copy1 has a correct CRC, then update copy2 including the CRC.  Then erase the CRC from copy1.  Then update copy1 including the CRC.
* If copy1 does not have a correct CRC, then update it including the CRC.  Then update copy2 including the CRC.

If the record isn't marked as "important" enough to require a backup copy, then the procedure is:

* Find the record, creating it if it doesn't exist.
* Erase the CRC first
* Update the record, and write the CRC.

### If the record is a counter

If the record is a counter, then no CRC is maintained, nor is a second copy.





