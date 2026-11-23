# usb2snes on the FX Pak

The FX Pak (sd2snes) has a USB port that shows up as a serial port —a CDC
ACM device, USB vendor `1209`, product `5a22`—, and its firmware speaks a
block protocol on it, usb2snes: read and write the SD card, read and write
the cartridge's memory while a game runs, boot a ROM, reset. The
`fxpak` tool of the-advocate (`snes/fxpak/`) speaks it directly; its README
has the commands.

What follows is read from the firmware's source, **sd2snes v1.11.0**,
`src/usbinterface.c` (the numbers in parentheses are its lines). Another
version may differ.

## Blocks

Everything is 512 bytes (`USB_BLOCK_SIZE`).

| bytes   | command                                      | response                     |
|---------|----------------------------------------------|------------------------------|
| 0-3     | `USBA`                                       | `USBA`                       |
| 4       | opcode                                       | 15, RESPONSE                 |
| 5       | space: 0 the SD card, 1 the FX Pak's memory  | error, nonzero when it failed |
| 6       | flags                                        | (INFO: features)             |
| 252-255 | size, U32 big-endian                         | size, U32 big-endian         |
| 256-    | address U32 big-endian, or a NUL-ended name  | (INFO: see below)            |

The opcodes are the enum order of `FOREACH_SERVER_OPCODE` (107-126): GET 0,
PUT 1, VGET 2, VPUT 3, LS 4, MKDIR 5, RM 6, MV 7, RESET 8, BOOT 9,
POWER_CYCLE 10, INFO 11, MENU_RESET 12, STREAM 13, TIME 14, RESPONSE 15.

After the response, data goes in 512-byte blocks, the last one padded: from
the FX Pak for GET and LS, to it for PUT. **PUT is not answered at its
end**: the next command is the sync, since the firmware reads data until
the size is met before it takes a command again.

* **INFO** puts the running ROM's path at 16, the firmware's version as a
  U32 at 256 and as a string at 260, and the device's name at 324 (692-707).
* **LS** answers size 1 and then entries: a type byte (0 directory, 1 file)
  and a NUL-ended name; a 2 means the next entry goes on in the next block,
  a 0xFF that the list is over (828-877). Names are code page 1252.
* **The memory space** is the FX Pak's own map (`memory.h`): the ROM's copy
  in PSRAM from `$000000`, the menu at `$C00000`, the save RAM at
  `$E00000`, the WRAM copy at `$F50000`, the menu's mailboxes at `$FF0000`.

## The paths that hang it

Some error paths of v1.11.0 never return, and the FX Pak then answers
nothing until the console is turned off with the USB cable out (with the
cable in it stays half powered):

* **GET of a file that does not exist**: the error is set but the size of
  the last `f_stat` is kept (474-477), and the read loop spins on an
  unopened file (763-770).
* **PUT of a file that cannot be opened**: the error is answered, but the
  data is still expected (278-281) and each block spins on `f_write`
  (297-307).
* **A transfer of 0 bytes** sends no data block (953), and the firmware
  ignores every command until the port is closed (409-417): this one a
  disconnect clears.
* **BOOT of a missing file** takes the console out of reset with nothing
  loaded (`memory.c` 249-253; 635-646).

A client has to look before it acts: list the parent directory before GET,
PUT or BOOT, and never ask for 0 bytes.
