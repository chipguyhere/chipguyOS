Copyright 2020 chipguyhere
License: GPLv3 (see /LICENSE for details)

To implement a 32-bit counter in EEPROM, we exploit the fact that "erasing" a byte sets it to 0xFF and that we can write bits
to clear them without an erase cycle.

A counter structure takes a multiple of 17 bytes of EEPROM that writes and maintains an incrementing 32-bit number in an
effort to spread writes and minimize erase cycles.  Each 16 bytes has a "control byte" that tracks how the other 4 bytes
are being used by the library

A design goal of the counter is that it stays consistent if there are "tears" (loss of power between successive byte writes).
We assume we have granularity to turn off bits in a single byte as an atomic operation,
as well as to erase a byte (set it to 0xFF) in a single operation.

## Control bytes

The lowest two bits of the first control byte describe the first word.  The two higher bits describe the next word, and so on.

* 11 = The word is the starting count before applying incrementers (for word[0]), or ignored as unused space (for any other word).
* 10 = The word is the starting count before applying incrementers (for word[1]), or a 32-bit-sized "large" incrementer (for any other word).
* 01 = Word is a "small" incrementer.
* 00 = Disregard the word. 

word[0] has precedence, and word[1] can only be the starting count when word[0]'s control bits are 00 (for disregard).

When the control byte is erased, it is 0xFF, and the first word becomes the count.  The first word (word[0]) is initialized to contain the
initial count.  The counter is consistent.  Then all remaining words are erased, as are their control bits.

Next, the first control byte is updated to 0b10110111 so the first word becomes an incrementer (its initial value of 0xFFFFFFFF represents zero
increment) and incrementers are enabled.


## Small incrementers

Small incrementers are for adding small numbers to the count without rewriting it.  Clearing bits adds some number to the count.  When an incrementer is erased, the value is 0xFFFFFFFF and has zero influence on the count.  It must be activated for use, and becomes active when its corresponding control bits are written with "01" to make it a "small incrementer".

In general, to add 1, clear the least significant bit that is still a 1.  0xFFFFFFFE adds one.  0xFFFFFFFC adds another one.

We can write bigger numbers to increment the count by starting from the most significant byte that is available.  Bytes are "available" when the counting of ones starting from the LSB has not consumed the most significant bit below the byte.

* A small number between 1 and 15 can be written in the lower nibble of any unused byte.  Invert it (XOR by 0xF) before doing so.
* Another number between 1 and 7 can be written in the upper nibble the same way.  The MSB of 1 indicates nibbles.
* A small number between 16 and 126 can be written to any unused byte.  Invert it (XOR by 0x7F) before doing so.  The MSB of 0 indicates a byte.

Generally speaking, the four bytes of each word will either contain ones, nibbles, or a byte value, deduced as follows:
* the last byte is reserved for counting ones, but can be used for a byte or nibble as long as doing so doesn't make use of the LSB, which must
be left untouched for recognition as a byte instead of a counter of ones.
* the third byte counts ones only when the last byte contains 0x00, otherwise it is counted as a byte or nibble.
* the second byte only counts ones when the last two bytes are 0x00 0x00.
* the first byte only counts ones when all remaining bytes are 0x00.

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

## 32-bit incrementers

An incrementer can be written that adds a 32-bit value to the count, overflowing back to zero (counters are always unsigned).  This is similar
to simply writing a new count, but by implementing this as adding a "difference", the existing incrementers do not need to be erased, and can
be added to by clearing more bits from them.

The value written must be inverted (value XOR 0xFFFFFFFF).  This ensures the default erased value represents zero.

## Writing a new incrementer

We will create a new incrementer when the existing incrementer(s) cannot hold an increase in count.

If the overflow is (3*126) or less, this can be written as a new "small incrementer".  Otherwise a "large incrementer" will be written,
which adds a 32-bit value to the count.

## When the incrementer space is exhausted
* If the starting count is at word[0]:
** Write the new count at word[1].
** Clear word[0]'s control bits to 00 so word[1] becomes the counter in effect.
** Erase all the incrementers.
** Continue below as the starting count is at word[1].
* If the starting count is at word[1]:
** Erase word[0] and copy the same starting count there.
** Erase all the control bytes (last to first) 
** When the first control byte is erased, word[0] now contains the active count.
** Update the first control byte to 0b10110111 so that incrementers are allowed again and the first one is enabled.










