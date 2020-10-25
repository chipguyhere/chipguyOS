Copyright 2020 chipguyhere
License: GPLv3 (see /LICENSE for details)

To implement a 32-bit counter in EEPROM, we exploit the fact that "erasing" a byte sets it to 0xFF and that we can write bits
to clear them without an erase cycle.

The counter requires EEPROM space of at least four 32-bit words, plus one control byte per four 32-bit words to describe
the contents of that word.

A design goal of the counter is that it stays consistent if there are "tears" (loss of power between successive byte writes).

## Control byte

The lowest two bits of the first control byte describe the first word.  The two higher bits describe the next word, and so on.

* 11 = Word is the count.  Applies only to the first word.
* 10 = Word is a component of the count
* 01 = Word is an incrementer to the count.
* 00 = Disregard the word.

When the control byte is erased, it is 0xFF, and the first word becomes the count.  The first word (word[0]) is initialized to contain the
initial count.  The counter is consistent and ready.

Next, word[1] is erased to 0xFFFFFFFF.  Then the control byte is set so the second word becomes an incrementer.  (0b11101111).

## Incrementers

Incrementers are for adding 1 to the count.  When erased, the value is 0xFFFFFFFF.  This adds zero.  After erasing it, the control byte is
updated to reflect that the word is now an incrementer.

To add 1, clear the least significant bit.  0xFFFFFFFE adds one.  0xFFFFFFFC adds two.

Sometimes there's a gap in writing the count and we need to write something larger than one.  Let's pretend we need to increment by 17
(0x11).  We will write it to the most significant byte.

```
  0xFFFFFFFF means zero
  0xFFFFFFFE means one
  0xFFFFFFFC means two
  0x11FFFFFC means nineteen
  0x11FFFFF8 means twenty.  We can continue to count ones by removing the LSB until it runs into the bit just before the 0x11.
```

Once the most significant byte is written, it's likely we may want to increment by that same count again (imagine the calling code is counting by tens).
If we detect this, we can turn off bits starting from the most significant bit, to indicate a repetition of the larger count.

```
  0x117FFFF8 means thirty-seven.  We've added seventeen a second time by turning one bit off (the F became a 7).
  0x113FFFF8 means fifty-four.  We did it again.
```

We can continue to burn the candle from both ends until we run out of bits.  When we do, we must start either a new incrementer, or write a new
count.

## Writing a new count

To write a new count, we select an unused word, write the count, verify it is written correctly, and then update the control byte to disregard
the words that are no longer part of the count.










