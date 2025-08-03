# armips-lzss
[![en](https://img.shields.io/badge/lang-en-red.svg)](https://github.com/leomontenegro6/mmbn-3-traducao-ptbr/blob/master/Asm/bn3plus/README.md)
[![pt-br](https://img.shields.io/badge/lang-pt--br-green.svg)](https://github.com/leomontenegro6/mmbn-3-traducao-ptbr/blob/master/Asm/bn3plus/README.pt-br.md)

This is a modified version of the [armips assembler](https://github.com/Kingcom/armips), originally made by Kingcom, and forked by a colleague of mine called denim to add new features.

The main change is the inclusion of a built-in LZSS 0x10 data compression routine, which basically allows compressing and inserting bios-compressed GBA data on-the-go. This is very useful for most GBA romhacks and translations out there, since there's no need to manually compress it with external tools.

# Usage

The tool usage is mostly the same as described in [the original repo](https://github.com/Kingcom/armips?tab=readme-ov-file#11-usage), with the addition of the new directive below:

```
.lz77gba "uncompressed-data-to-be-compressed.bin"
```

It works very similarly to the a `.incbin` call where you pass the path of a specific file to insert in the ROM, except that the data is compressed to LZSS 0x10 first, and only then it's inserted.

For example, imagine that a LZSS 0x10 compressed data is in the offset 0x7FC0D0 of a ROM. To insert the compressed data to that offset, this could do the trick:

```
.org 0x087FC0D0
    .lz77gba "Graphics/graphic-to-be-inserted-in-same-offset.bin"
```

But usually, compressed data are bigger than the original, so in this situation we might need to move the compressed data to the end of the ROM, and update its pointer accordingly. In that scenario, assuming the absolute pointer is in the offset 0x02262C, the data can be inserted this way:

```
; Storing pointer in the "graphic1" macro, for later use.
.org 0x0802262C
    .dw graphic1

; Inserting compressed data to the end of the ROM,
; and update its pointer accordingly.
.org 0x08800000
graphic1:
    .lz77gba "Graphics/graphic-to-be-inserted-in-the-end-of-the-rom.bin"
    .align
```

The example above will insert the graphic at 0x800000, and automatically update its pointer to point to the new place. After that, it'll align the file cursor to an address multiple of 4, to ensure other compressed graphics are going to be inserted in an even offset. Otherwise, bugs might occur.

And so on. If you have multiple compressed data to be inserted, just keep storing its pointers to other macros, and inserting them in the section below the pointer storage one, using multiple `.lz77gba` calls.

# Disclaimer

Although this tool is made available, it's provided as-is. The tool wasn't developed by myself and its source code isn't available, and no support will be provided besides this documentation. Use it at your own risk.