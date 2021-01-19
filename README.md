# ipaq-rx1950-rescue
**File collection for HP iPAQ rx1950 UART unbrick.**

What you'll need:
1. JTAG adapter
2. OpenOCD 0.3.1
3. PuTTY
4. Windows 2000/XP to use unsigned u-boot driver

Unbrick tutorial:

1. Disassemble the device and remove the motherboard.
2. Solder TMS, TDI, TDO, TCK and GND from the JTAG adapter to the motherboard (refer to img/ipaq-contacts.jpg and img/ipaq-schema.png) with 100Ω resistors. You may use an LPT<->JTAG adapter. Don't connect the iPAQ into JTAG yet!
3. Extract OpenOCD 0.3.1, extract giveio.zip in .\drivers and run install_giveio.bat.
4. Copy u-boot-rescue.bin and rx1950-u-boot.bin to C:\ drive.
5. Edit openocd.cfg in .\bin and paste the following data:

```
	interface parport
	parport_port 0
	parport_cable dlc5

	#jtag_device 4 0x1 0xf 0xe
	jtag_khz 100
	reset_config none

	source [find target/samsung_s3c2440.cfg]

	gdb_port 3333
	tcl_port 6666
	telnet_port 4444
	nand device s3c2440 s3c2440.cpu
```

6. Insert a FAT-formatted SD card with mtd0.dump.oob.0 and mtd1.dump.oob.0 in root into the iPAQ.
6.1. Plug in the iPAQ's data cable (with the charger).
7. Start OpenOCD and connect to 127.0.0.1:4444 via PuTTY.
8. Enter the following set of commands:
```
	halt
	soft_reset_halt
	bp 0x30080000 0x4 hw
	load_image C:\\rx1950-bootstrap.bin 0x0
	resume 0x0 (You should hit a breakpoint at address 0x30080000 here.)
	load_image C:\\u-boot-rescue.bin 0x31000000 (wait until the load is confirmed)
	resume 0x31000000 (An LED in the iPAQ should glow green) 
```
9. After the green LED comes on in the iPAQ, install the u-boot driver through Device Manager.
10. Open a serial connect via HyperTerminal on the USB port. Leave the connection settings at 9600 8n1.
11. Once you get the 'RX1950>' prompt, enter the following commands:
```
	mmcinit
	dynpart
	fatload mmc 1:1 0x31000000 mtd0.dump.oob.0
	nand erase 0x0 0x4000
	nand write.yaffs1 0x31000000 0x0 0x4200
	fatload mmc 1:1 0x31000000 mtd1.dump.oob.0
	nand erase 0x4000 0x40000
	nand write.yaffs1 0x31000000 0x4000 0x42000
```
11. fatload, nand erase and nand write.yaffs1 commands should return "OK".
12. Once the u-boot part is done, desolder the JTAG lines and remove power from the iPAQ.
13. Assemble the iPAQ and plug it into the computer again. You should get a 'Windows-Mobile based device' in Device Manager.
14.1. Proceed by flashing WM5.0 using the official HP flash SoftPaq.
14.2. Flash Freddom's Russian WM6.1 with WinHex (check the rus_wm61 folder for the tutorial) or use Victory144's WM6.1 flash package.