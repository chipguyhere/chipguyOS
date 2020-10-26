Copyright 2020 chipguyhere
License: GPLv3 (see /LICENSE for details)

To implement a 32-bit counter in EEPROM, we exploit the fact that "erasing" a byte sets it to 0xFF and that we can write bits
to clear them without an erase cycle.

The counter requires EEPROM space of at least four 32-bit words, plus one control byte per four 32-bit words to describe
the contents of that word.

A design goal of the counter is that it stays consistent if there are "tears" (loss of power between successive byte writes).
We assume we have granularity to turn off bits in a single byte as an atomic operation,
as well as to erase a byte (set it to 0xFF) in a single operation.

## Control byte

The lowest two bits of the first control byte describe the first word.  The two higher bits describe the next word, and so on.

* 11 = Word is the count, but do not apply incrementers yet.  (Their control bits are still being zeroed)
* 10 = Word is the count, and incrementers can now apply.
* 01 = Word is an incrementer.
* 00 = Disregard the word.

If more than one word is flagged as being the count, the lowest index word is the count.

When the control byte is erased, it is 0xFF, and the first word becomes the count.  The first word (word[0]) is initialized to contain the
initial count.  The counter is consistent and ready.

Next, word[1] is erased to 0xFFFFFFFF.  Then the control byte is set so the second word becomes an incrementer.  (0b11101111).

## Incrementers

Incrementers are for adding 1 and other small numbers to the count.  When erased, the value is 0xFFFFFFFF and has no influence on the count.

To add 1, clear the least significant bit that is still a 1.  0xFFFFFFFE adds one.  0xFFFFFFFC adds two.

We can write bigger numbers to increment the count by starting from the most significant byte that is available.  Bytes are "available" when the counting of ones has not consumed the most significant bit below the byte.

* A small number between 1 and 15 can be written in the lower nibble of any unused byte.  Invert it (XOR by 0xF) before doing so.
* Another number between 1 and 7 can be written in the upper nibble the same way.  The MSB of 1 indicates nibbles.
* A small number between 16 and 127 can be written to any unused byte.  Invert it (XOR by 0x7F) before doing so.  The MSB of 0 indicates a byte.

```
  0xFFFFFFFF means zero
  0xFFFFFFFE means one
  0xFFFFFFFC means two
  0x11FFFFFC means nineteen because we added seventeen.
  0x11FFFFF8 means twenty.  We can continue to count ones by removing the LSB until it runs into the bit just before the 0x11.
  0x11AFFFF8 means twenty-five.
  0x11AAFFF8 means thirty.
  0x118AFFF8 means thirty-two.  (Note how it was possible to borrow a bit from a nibble that had a two available)
```

We can continue to burn the candle from both ends until we run out of bits.  When we do, we must start either a new incrementer, or write a new
count.

## Writing a new count

To write a new count, we select an unused word, write the count, verify it is written correctly, and then update the control byte to disregard
the words that are no longer part of the count.










