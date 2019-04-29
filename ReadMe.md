# sentcode.py

A Python script to generate the secret codes needed to access landscapes in
Geoff Crammond's classic game: The Sentinel (aka The Sentry). It supports all
known versions of the game (BBC, C64, CPC, Spectrum, ST, PC (DOS), Amiga).

## Usage

Generate secret code for landscape 1234:
```
python sentcode.py 1234
```

Generate _all_ landscape codes (slow!):
```
python sentcode.py -a
```

For convenience, I've included the [full secret code list](sentinel_codes.txt).

## Secret Codes

Secret codes are created using values taken from the random number generator
(RNG) after generating a landscape. However, since the codes depend only on
the state of the RNG, we only need to know how many values were taken from the
RNG during the generation of each landscape. These counts were captured from
the original game and stored in the bundled `iterations.bin` file.

The original game supports 10000 landscapes (0000 to 9999). However the game
stores them in BCD format, meaning there are six additional hex landscapes
(A-F) for each ten original landscapes (0-9). These aren't readily accessible
in the original game, but optionally supported by [Augmentinel](https://simonowen.com/spectrum/augmentinel).

The script supports all 57344 possible landscape numbers (0000 to DFFF).
Landscapes E000 to FFFF are unavailable as they hang the landscape generator.
This is caused by a bias of 2 in the most significant nibble when determining
the number of sentries, leading to a loop exit condition that never passes.

## Random Number Generator

The RNG used is a 40-bit linear feedback shift register. Bits 33 and 20 are
XORed and fed back into bit 0 after each shift. The top 8 bits form the new
random value after 8 shift iterations.

The RNG is seeded from the 2-byte landscape number (already in BCD format),
which is placed in bits 0-15. Bit 16 is also set to ensure there is at least
one set bit in the state, as the XOR feedback doesn't provide a way to create
set bits from zero inputs.

## Generation Method

The script creates a secret code as follows:

- seed RNG using landscape number (as detailed above)
- read the required count of RNG calls from iterations.bin
- read 'count' RNG values to simulate landscape generation
- generate and discard 38 pairs of digits (part of code obfuscation!)
- generate 4 pairs of digits to give the final 8-digit secret code

## Random Digits

The secret codes are intentionally different between most versions of the game.
This is achieved by changing the routine that generates a pair of digits using
values from the RNG.

The BBC and C64 versions use a single 8-bit random value to generate two
digits. The upper nibble is used for the 10s digit and the low nibble for the
1s digit. If either value is hex (A-F) then 6 is subtracted to bring it back
into decimal digit range.

The CPC version uses a different RNG value for each digit, generating 1s before
10s. The Spectrum version does the same but in the opposite order.

There are only so many ways you can swap the nibbles around, so later versions
read and discard 3 values between each pair of digits. See the script code for
more details.

---

Simon Owen  
https://simonowen.com
