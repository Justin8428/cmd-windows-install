# Guide to installing Windows through the command line on modern UEFI GPT systems
## Why?
Because I couldn't find a single comprehensive guide to installing Windows through the command line I decided to write up a quick guide here. This is useful if you can't use the standard Windows installer for whatever reason (e.g. Windows RT, bypassing Windows 11 restrictions, etc)

Firstly, get to a command prompt (e.g. Shift + F10 in the Windows installer). Then

## Partition Disk
For modern UEFI GPT systems you can't just rely on a single bootable parition. You'll need to create at least 3 partitions in diskpart: ESR (boot information), MSR and your Windows partition (+ recovery if you want one, otherwise you'll need to use the setup disk if Windows can't boot).

Type diskpart to get into diskpart, then

### Setup GPT

 - `list disk` followed by `select disk #` to choose the appropriate disk
 - `clean` to wipe the disk
 - `convert gpt` **important** create a gpt partitioned drive, as Windows in UEFI will not boot off an MBR.
 - `create partition efi size=100` create efi boot partition
 - `format quick fs=fat32` format efi partition
 - `assign letter=s` assign it a letter in the WinPE environment, it will be removed upon Windows boot
 - `list disk` and `list partition` to check your work.

### Setup MSR
 - `create partition msr size=16` create MSR partition, no need to format

### Setup primary Windows partition
 - `create partition primary` create primary partition
 - `list partition` followed by `select partition #` to choose the appropriate partition (should be partition 3)
 - `format quick fs=ntfs` format 
 - `assign letter=c` assign it a letter in the WinPE environment, it will be removed upon Windows boot
 - `list disk` and `list partition` to check your work.

The partition layout should look like this:
![image](https://user-images.githubusercontent.com/24475790/174222251-6feba2b9-3ad1-4166-88a6-805fe729f20c.png)

Exit diskpart.

## Copy Windows Files and set up boot
Assuming your install drive is in drive D, the main partition is C, and the ESR is S (can use notepad --> file --> open to check), 
 - `dism /Get-WimInfo /WimFile:D:\Sources\install.wim` or `dism /Get-WimInfo /WimFile:D:\Sources\install.esd` to check what Index you want for the version of Windows you want to install, for example, 
![image](https://user-images.githubusercontent.com/24475790/174222787-fce2cafb-9d71-4edd-8e9c-42855faa684e.png)

Assuming you want to install Windows 11 Pro (index 6):
 - `dism /Apply-Image /ImageFile:D:\Sources\install.wim /index:6 /ApplyDir:C:\` to copy Windows Files
 - `bcdboot C:\windows /s S:` to copy boot files to ESR.

Reboot and the system should boot into Windows OOBE...
