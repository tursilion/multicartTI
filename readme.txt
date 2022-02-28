20160126

This program is designed to work using the 378 or 379 switching method up to 512k of ROM.

It will scan all banks, looking for cartridge headers. Any that are found, are added to a menu. The user selects from the menu, and the program sets that bank and jumps to the program.

It supports multiple program titles in the same executable, as well. The maximum number of entries is probably about 300, though I didn't test that high. (With one entry per bank, 512k gives 64 entries).

The program itself is multimenuC.bin -- if you load it by itself, it will pop up and give you an empty menu. (And it'll crash if you select A - it needs at least one item ;) ).

To load your cart in Classic99 you have to edit classic99.ini to use it, add these lines:

[usercart1]
name=MultiMenu
rom0=3|0000|80000|c:\work\ti\multicart\outcart3.bin

(if you already havea usercart1, use the next number, etc). This will make it show under the user menu.)

To make it work in Classic99 with User->Open, just make sure the filename ends with "3.bin".

multimenu.a99 is the source code, and is documented as much as I could as I went. :)

Now, to make the cartridge, I just concatenated files using the command prompt. Naturally you must ensure the files are exactly 8k (or padded to a multiple thereof) for this to work:

C:\WORK\TI\Multicart>copy /y /b AMBULNCC.BIN + /b ANTC.BIN + /b ANTEATC.BIN + /b BOXERC.BIN + /b RABBITTC.BIN + /b multimenuC.bin OUTCART.BIN
multimenuC.bin
AMBULNCC.BIN
ANTC.BIN
ANTEATC.BIN
BOXERC.BIN
RABBITTC.BIN
        1 file(s) copied.

(I didn't include the games - use any 8k games you have).

The trick to note here, is that multimenuC.bin MUST be in the powerup bank. In Classic99, that's the last bank on the ROM, so that is where I put it here. Note in that case you must pad to the full size of the EPROM (meaning if you put this on a 64k EPROM, you'd need to load 7 files before multimenuC.bin to make sure it was in the last bank). It's okay to duplicate the menu for padding purposes.

The technicals of how it works are this:

-First, before it does anything else, it looks from the top down to find a bank with a header of >AA71. This is assumed to be the menu! If multiple menus are present, they are expected to be identical (don't do that.) This lets it be moved around without needing to be recompiled. If it can't find itself, it will probably crash, I didn't put in error code.

-Next, it loads it's own internal character set containing graphics for the screen, thanks to Sometimes99er. It also sets the VDP registers to match the E/A values, sets the color table, loads the keyboard mode and clears the screen. 

-Next, it starts scanning banks, starting with the top of 512k (>607F), and going to the bottom (>6000). This one value would need to be changed for larger ROMs (there's really no reason it can't scan the full size of your new cart already...). Smaller EPROMs will just repeat their data, so this works nicely. It checkes the first two bytes of each bank for a header.

-If the header is >AA71, it assumes it is the menu program, and skips it. If it sees a header value of >AA3F, it also skips that one. This is so you can give other programs this header to hide them. I anticipate this will be useful when doing the 16k multicart.

-Next, if the first byte is not >AA, then there is no header at all, and it skips it.

-Assuming there is a header, it reads the program link address. If this is >0000, then there are no programs, and it skips it.

-Otherwise, we clear out a block in VDP to store the entry. The VDP entries are 32 bytes long and contain two bytes for the bank, two bytes for the program address, and 28 bytes for the program name - longer names will be truncated by the next entry. 

-We store the current bank ID, then copy out the program load address and name. If the name is less than 28 bytes, we have already padded the buffer

-Next we compare this buffer against all previous buffers, to check for duplicates (this is necessary since smaller ROMs will repeat their data). If this is not a duplicate, we save the buffer by incrementing the pointers, otherwise, we will reuse this buffer for the next program.

-We then check if there are any more programs, and loop till there are not

-we then decrement the bank, and as long as we are not finished (less than >6000), we loop back to start the header search again.

-When that is finished, we have a list of programs in VDP. We display the list on the screen using an alphabetic key. Only 22 characters of each name is displayed, and lowercase characters are forced to uppercase for display.

-When the user presses a key, it's checked for validity (is a letter, is in range). If so, we change the bank and jump to the start address of that program.

-The user may also use joystick 1 by pressing fire to bring up an arrow, moving the arrow up and down, and pressing fire again to select.

-You can change the page displayed by pressing a key 1-9 for the corresponding page (page number is ignored if it's invalid)

-The joystick can scroll the list by pressing and holding up or down at the top or bottom of the screen

-At this point, the bank is selected.. so pressing QUIT will still bring up the same program in the menu. This is why some way to reset the 379 is needed. :) 

-Added GROM support to 0.4 - this scans in pretty much the same way as the ROMs do, but from GROM bases >9800 to >983C, all GROMs from >0000 to >E000. You can make a one-line change to skip the console GROMs

If you wanted to see the list... bring up Classic99 and the debugger, set the VDP watch address to >1200, and you can watch the program list being built. Easiest way: get the multicart loaded, go to the TI selection menu, and before you press '2 for MULTICART', open the debugger. Select the VDP radio button, enter 1200 and press Apply. This will show you VDP memory at >1200. Then press '2' to load the multicart, and you will be able to see the menu data - the first two bytes are the bank index, the next two are the load address, the rest is the name. ;)

-Version 105 makes launching of GPL software more reliable and is the first version on my website

