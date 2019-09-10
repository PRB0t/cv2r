# cv2r - Castlevania II: Simon's Quest Randomizer [![Build Status](https://travis-ci.org/BloodSweatAndCode/cv2r.svg?branch=master)](https://travis-ci.org/BloodSweatAndCode/cv2r)

cv2r - Castlevania II: Simon's Quest Randomizer by **BloodSweatAndCode**. Come find me running this and other retro games on [Twitch](https://www.twitch.tv/bloodsweatandcode)!

**WARNING** This is still **pre-alpha**

## usage

```
Castlevania II: Simon's Quest Randomizer by BloodSweatAndCode

Usage
  cv2r [options] [vanilla_rom_file]

Examples
  # Show this help
  node .\bin\cv2r --help

  # Generate a rom with the seed "BSAC" at standard difficulty
  node .\bin\cv2r --seed BSAC cv2.nes

  # Generate a rom with the seed "BurbAndSath" at hard difficulty
  node .\bin\cv2r --seed BurbAndSath --difficulty hard cv2.nes

  # Generate a rom with a random seed using the "Rondo of Burb" palette
  node .\bin\cv2r --palette rondo-of-burb cv2.nes

Options
  -V, --version                  output the version number
  -d, --difficulty <difficulty>  Difficulty setting: easy, standard, hard
  -j, --json                     output patch and spoiler as json, disables all other output
  -o, --output <output>          filepath for randomized rom output
  -p, --palette <palette>        palette selection: rondo-of-burb, simons-quest
  -r, --run                      immediately run with emulator after randomizing (fceux or OpenEmu must be in PATH)
  -s, --seed <seed>              seed to use for randomization
  -z, --debug                    enable debug output
```

## install

I plan to make executables so this installation isn't necessary, but until then, follow these steps:

1. Install [node.js](https://nodejs.org/en/)
2. Download the latest build of [cv2r](https://github.com/BloodSweatAndCode/cv2r/archive/master.zip)
3. Extract cv2r to a folder
4. Copy a vanilla cv2 rom into the cv2r folder as "cv2.nes"
5. In a command prompt:
```
cd /path/to/cv2r
npm install

# on Windows, "BSAC" is replaced by your seed (any string)
node --no-warnings .\bin\cv2r --all --seed BSAC cv2.nes

# on Mac/Linux (I've done ZERO Linux testing), "BSAC" is replaced by your seed (any string)
./bin/cv2r --all --seed BSAC cv2.nes
```

You can now find your new cv2 rando rom in the `/path/to/cv2r/tmp` folder

## developer notes

### Screen dimensions

Screen width, in tiles, is always a multiple of `0x10` and height is a multiple of `0x0E`. The maximum width and height is 4 screens, with the only screen actually being `0x40`x`0x38` is Castlevania.

| # of screens | width | height |
|--------------|-------|--------|
| 1            | 0x10  | 0x0E   |
| 2            | 0x20  | 0x1C   |
| 3            | 0x30  | 0x2A   |
| 4            | 0x40  | 0x38   |

### Death Fight

`01:8D1C` execute breakpoint with X==#7

$32A death y position
$32B-$32D death scythe y position
$34E death x position
$34F-$351 death scythe x position
$372 Death y delta
$373 holds the next x pos increment of the scythe
$384 Death y delta????
$396 Death's x delta
$397-$399 Death scythes' x delta
$426 - 1 if death mirrored, 0 if not

`07:E1B7`,`1E18B` code where value is added to scythe x position

### sprite mapping

| sprite | location |
|--------|----------|
| death drop (golden knife) | $29518-$2952B |
| camilla drop (cross, 1st half) | $2A618-$2A619 |
| camilla drop (cross, 2nd half) | $2A71E-$2A71F |

### RAM mapping

| address | function                      |
|---------|-------------------------------|
| 001C    | currently loaded memory bank  |
| 003F    | 0xFF when talking to NPC      |
| 003BA-003BF | 49 - golden knife (Death drop) |

### bank switching

Set value $1C to the number of the bank then execute a JSR at the following location. If you want to set it based on the accumulator, subtract 2 from the RAM address, which will in turn set $1C to the value of the accumulator.

| bank | RAM address |
|------|-------------|
| 7    | $C185       |

### inventory

$4F is selected quest item
$90 is selected weapon/carry item

### code locations for all actors that can hold items

| name                           | RAM  | ROM   | $7F       | notes              |
|--------------------------------|------|-------|-----------|--------------------|
| merchant (weapon/item)         | EDD8 | 1EDE8 | see below |                    |
| merchant (whip)                | EDF4 | 1EE04 | see below |                    |
| crystal dude (blue)            | 906F | 507F  | 0x55      |                    |
| crystal dude (red)             | 9088 | 5098  | 0x56      |                    |
| orb                            | 8794 | 47A4  | 0x18-0x1C |                    |
| laurel dude (laruba)           | 9347 | 5357  | 0x78      |                    |
| flame whip dude                | 8C72 | 4C82  | 0x0E      |                    |
| diamond dude                   | AA3A | 6A4A  | 0x12      |                    |
| secret merchant (silver knife) | AE12 | 6E22  | 0x10      |                    |
| secret merchant (silk bag)     | AE07 | 6E17  | 0x0F      |                    |
| Death                          | 87C7 | 47D7  | 0x77      | still appears as knife no matter what item it actually gives you |
| Camilla                        | 87BF | 47CF  | 0x26      | still appears as cross no matter what item it actually gives you |
| sacred flame                   | 87CD | 47DD  | 0x76      | still appears as flame no matter what item it actually gives you |

| merchant      | $7F  |
|---------------|------|
| thorn whip    | 0x33 |
| chain whip    | 0x34 |
| morning star  | 0x35 |
| white crystal | 0x32 |
| holy water    | 0x37 |
| dagger        | 0x36 |
| garlic        | 0x2F |
| laurels       | 0x30 |
| oak stake     | 0x1D |

; if (($7F == 0x26) && ($7F < 0x1D || $7F > 0x37))  {
;   $600E = 0x01
; }

### sram usage

We use SRAM values ($6000-$7FFF) to track progressive whip and crystal upgrades throughout the game to ensure you can't get the same progression item from the same actor more than once. It's also used for other state and temporary variables when necessary. Here's how we use those values.

| memory | usage |
|--------|-------|
| $6000  | Temporary variable used to store original X register when loading the correct (with respect to progression) crystal sale icon for merchants |
| $600C  | Used by is-checked. A value of 1 means the current actor has already been checked. 0 means it has not and an item can still be acquired. |
| $600D  | Temporary variable used to store original memory bank when bank switching is necessary, specifically in the game state checks for jovah warp and inventory deselect |
| $600E  | If set to a non-zero value, it indicates that an actor should use its normal, non-refuse text despite having been marked as checked. This is necessary because most non-merchant actors add the item to your inventory _before_ they display their text. |
| $600F  | If set to 0x01 a jovah warp is currently in progress |
| $6010-$601B | Tracks whip progression. When you acquire a whip, it adds an entry to this range which marks the actor as "checked". The range consists of 3 entries, 4 bytes in size each. The 4 bytes, in order, are: objset($30), area($50), submap($51 & 0x7), actor identifier($7F) |
| $601C-$6024 | Tracks crystal progression. When you acquire a crystal, it adds an entry to this range which marks the actor as "checked". The range consists of 2 entries, 4 bytes in size each. The 4 bytes, in order, are: objset($30), area($50), submap($51 & 0x7), actor identifier($7F) |

### unused but interesting values

(ROM) 0x1EE25 is 0x092010, which seems to indicate it's a carryable item, but 0x10 doesn't match up with the bit flags currently mapped.

| carry item  | bit |
|-------------|-----|
| silk bag    | 1   |
| magic cross | 2   |
| laurels     | 3   |
| garlic      | 4   |

### dialog text

`0xCC84` is where all the text starts in the ROM. First one is "What a horrible night to have a curse".

| Char | Value   |
|------|---------|
| (space) | 0x00 |
| A    | 0x01    |
| B    | 0x02    |
| C    | 0x03    |
| D    | 0x04    |
| E    | 0x05    |
| F    | 0x06    |
| G    | 0x07    |
| H    | 0x08    |
| I    | 0x09    |
| J    | 0x0A    |
| K    | 0x0B    |
| L    | 0x0C    |
| M    | 0x0D    |
| N    | 0x0E    |
| O    | 0x0F    |
| P    | 0x10    |
| Q    | 0x11    |
| R    | 0x12    |
| S    | 0x13    |
| T    | 0x14    |
| U    | 0x15    |
| V    | 0x16    |
| W    | 0x17    |
| X    | 0x18    |
| Y    | 0x19    |
| Z    | 0x1A    |
| .    | 0x1B    |
| '    | 0x1C    |
| ^    | 0x1D    |
| ,    | 0x1E    |
| 0    | 0x36    |
| 1    | 0x37    |
| 2    | 0x38    |
| 3    | 0x39    |
| 4    | 0x3A    |
| 5    | 0x3B    |
| 6    | 0x3C    |
| 7    | 0x3D    |
| 8    | 0x3E    |
| 9    | 0x3F    |
| (new line) | 0xFE |
| (end) | 0xFF   |

### in lib/core.js

* **Dead River - Part 2** and **Dead River to Brahm** share the same actors, so this needs to be accounted for during any randomizing that involves the actors in those screens. Essentially, unless otherwise handled, **Dead River - Part 2** will take precedence since it is defined after **Dead River to Brahm** in the `core.js` definition file.
* `npc.crystalDude` is the same as the (secret) merchant in the non-town pattern tables

### pattern table re-mapping

I had to store new 1 byte values. High 3 bits are zero (but could be bg table), low 5 bits are sprite table for every screen in the game. These needed to stored in a space that could be accessed mathematically based on 3 values: objset ($30), area ($50), and submap ($51 AND #$0xF). Those 3 values together compose a unique reference to every screen. The table below shows how I allocated free bytes in the ROM to essentially store a multi-dimensional array.

| objset | # of areas | # of submaps | map storage |
|--------|------------|--------------|-------------|
| 0x00   | 24         | 2            | 48          |
| 0x01   | 11         | 4            | 44          |
| 0x02   | 10         | 4            | 40          |
| 0x03   | 5          | 5            | 25          |
| 0x04   | 4          | 2            | 8           |
| 0x05   | 1          | 2            | 2           |

So I'm mapping 167 bytes, only using 93 of those bytes in total, thereby wasting 74 bytes as unused space between the valid values of the mult-dimensional array. The pseduo-code calculation below shows roughly how, stored in this manner, I can access any bg/sprite table index at any time with the objset, area, and submap values:

```
// OBJ_OFFSET is 0x30, as there's equal space between each objset in the stored bytes.
(objset * OBJ_OFFSET) + (area * num_of_submaps_for_this_area) + submap
```

With the values stored, we then hijack the typical rudimentary pattern table mapping code. We write to unused ROM space at `0x7860` the compiled opcodes of the 6502 asm contained in the file at `data/pattern.asm`. This code is responsible for executing the calculation above and then properly selecting a sprite table index based on it. All we need to do after that is overwrite 3 particular bytes of code to point to our new code. Essentially we replace the code at `0x1CCDF` with the following:

```
JSR $B850: 20 50 B8
```

and voila! Each screen now has it's own pattern table mapping! Here's the original mapping, which shows there were only 6 options (instead of the 93 possible options we have now). Here we can see what background and sprite table indexes correspond to what enemy groupings.

objset | pattern pointers | bg   | sprite | name
-------|------------------|------|--------|--------
0      | 0x1CCF9          | 0x00 | 0x01   | town
1      | 0x1CCFB          | 0x08 | 0x09   | mansion
2      | 0x1CCFD          | 0x02 | 0x03   | woods 1
3      | 0x1CCFF          | 0x04 | 0x05   | woods 2
4      | 0x1CD01          | 0x06 | 0x07   | graveyard
5      | 0x1CD03          | 0x0B | 0x0C   | castlevania

oak
----
0x03 woods1 fishman spitting head
0x05 woods2 flames???
0x07 graveyard oprn flower

### random notes

In RAM, `7:E373` is a subroutine that does 4 LSRs. `7:E378` does 4 ASLs.


red crystal tornado at 01:A956 ram, 6966 rom
core function 07:C0E7

// merchant purchasable items: 0x1ee22-0x1ee39

// code for blue crystal guy at 1:906F(RAM) 0x507F(ROM)

// code for red crystal guy at 1:9088(RAM) 0x5098(ROM)

// 07:C08A function for red crystal


// F7 is dpad

// 0x1F6AD is where the dialog starts, 0x0E 0x00 means 0x00 14 times

// 0x83CC code specific to merchants?
// 0x58,0x59 map -- 0x5E,0x5F ???
// 0x80EC - code that stores npc "type", value at 0x9140 for thorn

// 0x3ba is 0x2E which is the lower 7 bits of 0xAE which is the actor "type"
// 0x4da is the actor "value" 0x08
// 0x34e is 0xC0
// 0x32a is 0xC0
// 0x3de is 0x08
// 0x438 is 0x01

// $1F6F7 is the price line in dialog ROM

// 0x414 is 0xOB after a pointer chek on 0x3ba when entering room
// 0x414 times 3 (0x21) goes into 0x94,
// then ised to point to 0xDDC3 = (01 1e 0c)
// 1e goes into 0x306, 0c goes into 3f0
//

// $40 is 0x28, gets set to 0x93
// 0x888D is where whip is checked for merchant chat

// $1ED3D in ROM is 0x5B, which is the whip icon, gets stored at $703
// the format is 3 bytes for each entry. The first byte is the
// icon for sale (index for background PPU table). The second and third
// bytes are the price. PRICE RANDO!
// prices: 0x1ED25-0x1ED45
// price high byte gets stored at $29, $93
// price low byte gets stored at $28

// 0xE09C is where 0x3de AND 0xf7, which excludes only values with the 4th bit
// set, meaning merchants that sell whips. This gets put in 0x3de, or zero

// $f6e7 for line in dialog for dashes and hearts
// 0x1EFE2 is where yes/no dialog choice happens (0xEFD2 RAM)

// $E08C (RAM) watch me whip?

// $40 and $93 determine icon for sale
// $1D is 0x2E, $18 is 0x05

// $7A
// * 0-3 == presenting dialog background
// * 4 == printing dialog text
// * 5 == non-interactive text end, waiting for B press
// * 6 == waiting for yes/no choice

// $7E
// * 0 == start menu
// * 1 == no interaction dialog
// * 2 == yes/no dialog

// $F1 is dpad

// ECC7 is where we move to the whip icon, C5D0 is where pointer is set
// EE9D is the dialog text loop

// 7A increments after dialog completes (5), i think it managed dialog state
// does $164 hold this value at any stage?

// C058 checks $1A, if Z flag not set, skips all merchant logic!

// $7A needs to be 0x01 for normal dialog

// $103???? at C)FB

// Change $3BA (or whatever near location has the merchant 0x2E) to the value 0x35.
// The merchant then moves like a normal NPC and will deliver an npc dialog with no
// yes/no interaction. Dialog now points to the unused "you level has increased..."
// messages, which should be easy to change.

// Using $7F as a unique ID for actors doesn't work because the multiple
// laurel and garlic merchants share the same value. Additionally, laurel
// merchants have a $7F value of 0x00, which is intended to be an empty value
// in the current progressive whip upgrade logic, so that will have to
// change.

// whip sale code 0xEDF4 RAM, 0x1EE04 ROM

// palette hacking
  // $82 day/night? 1/0
  // $C7E7 is where day/night is checked

800E, 401e (rom) for the area checks to determine which actor to load

3d and 3e are used as a pointer to the actors for a particular area