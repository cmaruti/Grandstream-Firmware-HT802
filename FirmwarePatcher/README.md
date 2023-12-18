# GrandStream Firmware Patcher
A tool to extract, decrypt and patch [firmware](http://www.grandstream.com/support/firmware)  

With this tool you can create Custom Firmware for Grandstream devices  

## Usage
### Info
```bash
$ ./GSFW.py info -i ht802fw.bin
** Firmware Info **
Contained files:
	 ht802boot.bin 	version: 1.0.9.1 	size: 245760 bytes
	 ht802core.bin 	version: 1.0.9.1 	size: 1187840 bytes
	 ht802base.bin 	version: 1.0.9.2 	size: 2838528 bytes
	 ht802prog.bin 	version: 1.0.9.3 	size: 3223552 bytes
```  

```bash
$ ./GSFW.py info -i ht801fw.bin
** Firmware Info **
Contained files:
         ht801boot.bin  version: 1.0.51.1       size: 245760 bytes
         ht801core.bin  version: 1.0.51.1       size: 1269760 bytes
         ht801prog.bin  version: 1.0.51.1       size: 3371008 bytes
         ht801base.bin  version: 1.0.51.1       size: 2994176 bytes
         ht801boot.dvf9919.bin  version: 1.0.51.1       size: 249856 bytes
         ht801core.dvf9919.bin  version: 1.0.51.1       size: 1273856 bytes

``` 

### Extract 
Decrypt and extract all contained files
```bash
$ ./GSFW.py extract -i ht802fw.bin -d /tmp/test -k 37d6ae8bc920374649426438bde35493
** Firmware Extract **
Used key: 37d6ae8bc920374649426438bde35493
Extracting files:
	 /tmp/test/ht802boot.bin 	version: 1.0.9.1 	size: 245760 bytes
		Head key: 738d0cb8bc02736494244683fb5e4539
		Body key: 000b06d307e2041a0a0bfd2100010000
		Decrypting...
	 /tmp/test/ht802core.bin 	version: 1.0.9.1 	size: 1187840 bytes
		Head key: 738d0cb8bc02736494244683fb5e4539
		Body key: 000c834507e2041a0a0bfd2100010000
		Decrypting...
	 /tmp/test/ht802base.bin 	version: 1.0.9.2 	size: 2838528 bytes
		Head key: 738d0cb8bc02736494244683fb5e4539
		Body key: 000d64c807e2041a0a0bfd2100010000
		Decrypting...
	 /tmp/test/ht802prog.bin 	version: 1.0.9.3 	size: 3223552 bytes
		Head key: 738d0cb8bc02736494244683fb5e4539
		Body key: 000ee4a507e2041a0a0bfd2100010000
		Decrypting...
```

```bash
$ ./GSFW.py extract -i ../../ht801fw.bin -d /tmp/test
** Firmware Extract **
Used key: 37d6ae8bc920374649426438bde35493
Extracting files:
         /tmp/test/ht801boot.bin        version: 1.0.51.1       size: 245760 bytes
                Head key: 738d0cb8bc02736494244683fb5e4539
                Body key: 000b31d607e709160d1dfd2000010000
                Decrypting...

         /tmp/test/ht801core.bin        version: 1.0.51.1       size: 1269760 bytes
                Head key: 738d0cb8bc02736494244683fb5e4539
                Body key: 000ca1d507e709160d1dfd2000010000
                Decrypting...

         /tmp/test/ht801prog.bin        version: 1.0.51.1       size: 3371008 bytes
                Head key: 738d0cb8bc02736494244683fb5e4539
                Body key: 000e60c907e709160d1dfd2000010000
                Decrypting...

         /tmp/test/ht801base.bin        version: 1.0.51.1       size: 2994176 bytes
                Head key: 738d0cb8bc02736494244683fb5e4539
                Body key: 000d138507e709160d1dfd2000010000
                Decrypting...

         /tmp/test/ht801boot.dvf9919.bin        version: 1.0.51.1       size: 249856 bytes
                Head key: 738d0cb8bc02736494244683fb5e4539
                Body key: 000bdf5107e709160d1dfd2000010000
                Decrypting...

         /tmp/test/ht801core.dvf9919.bin        version: 1.0.51.1       size: 1273856 bytes
                Head key: 738d0cb8bc02736494244683fb5e4539
                Body key: 000c4b5907e709160d1dfd2000010000
                Decrypting...
```  

You can now extract filesystem from sqfs:  
`$ sudo binwalk -e /tmp/test/ht802prog.bin`  
Make you mods...  
Compile mksquashfs from https://squashfs-lzma.org/  
And repack the sqfs:  
`$ sudo ./mksquashfs squashfs-root progmod.squashfs -comp xz -all-root -noappend -always-use-fragments `

### Patch  
Replace the body of a modified file (eg: modified prog squashfs)  
```bash
./GSFW.py patch -i ht802fw.bin -o ht802fw.bin.mod -n ht802prog.bin -b progmod.squashfs -v 1.0.9.4 -k 37d6ae8bc920374649426438bde35493
** Firmware Patch **
Used key: 37d6ae8bc920374649426438bde35493
Looking for file: ht802prog.bin
	File found!
	Decrypting file header:
		Head key: 738d0cb8bc02736494244683fb5e4539
		Body key: 000ee4a507e2041a0a0bfd2100010000
		Decrypting...
	New version:   1.0.9.4
	New file size: 3223552 bytes
	New checksum:  0xe4a5
	Patching file header...
	Encrypting new file:
		Head key: 738d0cb8bc02736494244683fb5e4539
		Body key: 000ee4a507e2041a0a0bfd2100010000
		Encrypting...
Patching firmware header...
Writing new firmware
```  
# Firmware format
## Header (648 bytes)
| Size (byte)  | Type | Name | Description |
| :----------: | ---- | ---- | ------- |
| 4 | Unsigned LE Int | Magic | 0x23C97AF9 |
| 64 | Char string | File name 1 | ht802boot.bin |
| 64 | Char string | File name 2 | ht802core.bin |
| 64 | Char string | File name 3 | ht802base.bin |
| 64 | Char string | File name 4 | ht802prog.bin |
| 64 | Char string | File name 5 | empty |
| 64 | Char string | File name 6 | empty |
| 64 | Char string | File name 7 | empty |
| 4 | Unsigned LE Int | File size 1 | length in bytes |
| 4 | Unsigned LE Int | File size 2 | length in bytes |
| 4 | Unsigned LE Int | File size 3 | length in bytes |
| 4 | Unsigned LE Int | File size 4 | length in bytes |
| 4 | Unsigned LE Int | File size 5 | length in bytes |
| 4 | Unsigned LE Int | File size 6 | length in bytes |
| 4 | Unsigned LE Int | File size 7 | length in bytes |
| 4 | Unsigned LE Int | File version 1 | 0x01000a02 --> 1.0.10.2 |
| 4 | Unsigned LE Int | File version 2 | 0x01000a02 --> 1.0.10.2 |
| 4 | Unsigned LE Int | File version 3 | 0x01000a06 --> 1.0.10.6 |
| 4 | Unsigned LE Int | File version 4 | 0x01000a07 --> 1.0.10.6 |
| 4 | Unsigned LE Int | File version 5 | empty |
| 4 | Unsigned LE Int | File version 6 | empty |
| 4 | Unsigned LE Int | File version 7 | empty |
| 4 | Unsigned LE Int | File mask 1 | 0x00000000 |
| 4 | Unsigned LE Int | File mask 2 | 0x00070000 |
| 4 | Unsigned LE Int | File mask 3 | 0x00000001 |
| 4 | Unsigned LE Int | File mask 4 | 0x00000000 |
| 4 | Unsigned LE Int | File mask 5 | 0x00000000 |
| 4 | Unsigned LE Int | File mask 6 | 0x00000000 |
| 4 | Unsigned LE Int | File mask 7 | 0x00000000 |
| 112 | Byte array | Unknown | All zeros |

## Body
| Size (byte)  | Type | Name | Description |
| :----------: | ---- | ---- | ------- |
| File size 1 | Byte array | File 1 | Encrypted (header + body) (ht802boot.bin, uboot)|
| File size 2 | Byte array | File 2 | Encrypted (header + body) (ht802core.bin, linux kernel)|
| File size 3 | Byte array | File 3 | Encrypted (header + body) (ht802base.bin, squashfs, root fs)|
| File size 4 | Byte array | File 4 | Encrypted (header + body) (ht802prog.bin, squashfs, mounted on /app)|
| File size 5 | Byte array | File 5 | Encrypted (header + body) |
| File size 6 | Byte array | File 6 | Encrypted (header + body) |
| File size 7 | Byte array | File 7 | Encrypted (header + body) |

# File format
| Size (byte)  | Type | Name | Description |
| :----------: | ---- | ---- | ------- |
| 512 | Byte array | Header | Encrypted only first 32 bytes |
| File size - 512 | Byte array | File | Encrypted (by block of 32 bytes each time reinitializing the cipher) |

# File encryption
Each file is encrypted with AES 128 CBC  
The default key is: 37d6ae8bc920374649426438bde35493 (16 bytes converted in byte array)  
The IV is: Grandstream Inc. (16 bytes)  
Only the first 32 bytes of the header are encrypted  
All the body is encrypted by block of 32 bytes, each time reinitializing the cipher (LOL)  

They wrongly convert the key from hex to bytes:  
They always subtract 0x37 from hex 'A'-'F', also if they are lowercase, so 'a'-'f' will become 0x2a...0x2f instead of 0xa...0xf  
Infact, if you try to decrypt the files using the default key but uppercase, the decryption will fail!  

The header key is the given key with all the pair of nibble swapped  
The body key is formed by the second block of 16 bytes of the decrypted header, with all the pair of bytes swapped  

# Find encryption key
The "new_provision" binary search the key in these two places:  
- in nvram at variable 242: `nvram get 242` (empty if not present)  
- in a viartual file: `cat /proc/gxp/dev_info/hw_features/default_fwkey` (nokey if not present)  

If in both places there is no key, the binary falls back on two standard keys:  
- 37d6ae8bc920374649426438bde35493  
- de395fe3e08d8a47984de86e36b37eb0  

# Supported devices
This tool was tested with the following devices, other devices may also be supported.  
- HT801
- HT802
- HT814
- GXP16XX (using [GS_NUM_FILES](https://github.com/BigNerd95/Grandstream-Firmware-HT802/blob/master/FirmwarePatcher/GSFW.py#L11) = 8)
