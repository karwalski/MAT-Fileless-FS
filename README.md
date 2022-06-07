# MAT-Fileless-FS
A fileless filesystem, used for raw logging on SD cards without need for filesystem libraries

## Version 1.1
This is a branch is a re-write of the original, which was designed in April 2022 and lost on a corrupted hard drive. Python files for formatting a SD card, and for reading values to a csv file will be recreated and uploaded to this repo.

This filesystem standard was created to solve the problem of having to use HAL libaries to manage a FAT file system of an SD card being addressed via SPI on a Nucleo STM32 microcontroller. It is designed to be compatible with the Master Boot Record and partition structure of modern filesystems.

The first block (512 bytes) of the storage device is used to record the MBR. The second block is used to record a Partition Boot Record (PBR). Another block is used to record the current segment to be used for writting a rotating raw loggin data (usually the third, however programmable in the PBR to avoid too many writes damaging a solid state block).

The allocation of 6F is used as the partition ID. It is believe that this ID is otherwise unused (https://en.wikipedia.org/wiki/Partition_type)

Master Boot Record (MBR)
|	Address (dec)	|	Address (hex)	|	Size (bytes)	|	Value (Hex)	|	Description	|
|	---	|	---	|	---	|	---	|	---	|
|	0	|	0x0000	|	440	|	0x00	|	unused bootstrap code	|
|	440	|	0x01B8	|	4	|	0x0054414D	|	Signature = MAT (little-endian)	|
|	444	|	0x01BC	|	2	|	0x0000	|	No copy-protection	|
|	446	|	0x01BE	|	1	|	0x80	|	Partition 1 Active	|
|	447	|	0x01BF	|	3	|	0x000000	|	unused	|
|	450	|	0x01C2	|	1	|	0x6F	|	Partition ID (6F = MAT)	|
|	451	|	0x01C3	|	3	|	0x000000	|	unused	|
|	454	|	0x01C6	|	4	|	0x01000000	|	First LBA (PBR location)	|
|	458	|	0x01CA	|	4	|	[(size of card / 512 bytes) - 1]	|	Available blocks (limited to 2 TB)	|
|	462	|	0x01CE	|	48	|	0x00	|	3 additional partition entries for extending beyond 2TB (same structure as 446-461)	|
|	510	|	0x01FE	|	4	|	0x55AA	|	Boot record identifier	|

Partition Boot Record (PBR)
|	Address (dec)	|	Address (hex)	|	Size (bytes)	|	Value (Hex)	|	Description	|
|	---	|	---	|	---	|	---	|	---	|
|	0	|	0x0000	|	87	|	0x00	|	unused	|
|	87	|	0x0057	|	3	|	0x54414D	|	Signature = MAT (little-endian)	|
|	90	|	0x005A	|	4	|	0x02000000	|	Next segment to be used in the rotation	|
|	94	|	0x005E	|	4	|	example 65,536	|	Segment size. Number of 512 byte blocks per segment	|
|	98	|	0x0062	|	412	|	0x00	|	unused	|
|	510	|	0x01FE	|	4	|	0x55AA	|	Boot record identifier	|

The logging of raw data can then be done in a minimal approach, for example recording 56 sensor values each 9 bytes in length.
Unix time - 4 bytes
Sensor id - 1 byte
Sensor value - 4 bytes

The system should remember the segment length, and update the PBR at the desired interval maintaining the integrity of the PBR by not executing too many writes on solid state devices.
