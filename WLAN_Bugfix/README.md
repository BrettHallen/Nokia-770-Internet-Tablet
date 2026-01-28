# Bug 2006 - Memory corruption during WLAN use
Reported by [Tilman Vogel](https://web.archive.org/web/20120216094424/https://bugs.maemo.org/show_bug.cgi?id=2006).<br>

## Summary
Depending on the memory location to which the modules ```umac.ko``` and ```cx3110x.ko``` get loaded, exactly two consecutive bytes at fixed physical locations in memory get overwritten by zeroes everytime there is WLAN activity: while scanning for networks as well as when data is actually transmitted.

On a vanilla NOKIA770_2006SE_3.2006.49-2_PR_MR0, the modules get loaded at (```cat /proc/modules```):<br>
- cx3110x 51420 0 - Live 0xBF03F000
- umac 253316 1 cx3110x, Live 0xBF000000

In this case, the two bytes are at physical location 0x1304B8B4 and 0x1304B8B5 (these addresses include an offset of 0x10000000 - see ```/proc/iomem```).<br>

When booting the same OS from an ext2 formatted MMC partition, then the modules are at:<br>
- cx3110x 51420 0 - Live 0xBF04E000
- umac 253316 1 cx3110x, Live 0xBF00F000
- ext2 43524 1 - Live 0xBF003000
- mbcache 7716 0 - Live 0xBF000000

Due to the two extra modules, ```umac.ko``` and ```cx3110x.ko``` are shifted by 0xF000. Accordingly, the corrupted bytes get shifted by 0xF000 to 0x1305A8B4 and 0x1305A8B5.<br>

## Installing
- Copy the .DEB package to MMC card
- Ensure WLAN is off 
- Run the package from the MMC
- Reboot
- After installation there should be additional ```bugfix``` line in kernel logs:
``` 
$ dmesg | grep -i 'cx3'

Unloaded CX3110x  driver, version 0.8
CX3110x chip variant: STLC4370
CX3110x: firmware version: 2.13.0.0.a.13.14
Loaded CX3110x driver, version 0.8.1-bug2006-fix1
```
