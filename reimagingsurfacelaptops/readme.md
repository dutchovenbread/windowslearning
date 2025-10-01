# Reimaging Windows laptops

## Initial effort -- Download from Windows Surface BMR page
The following site is supposed to provide recovery images for Windows Surface machines:

https://support.microsoft.com/en-us/surface-recovery-image

Late in September 2025 it was not provididng the images. Putting in the serial number yielded an error.
## Second effort -- Vanilla windows 11 image
Downloaded the following file:

Win11_24H2_English_Arm64.iso

This is a plain Windows 11 image. The laptop would try to boot off of this. This was good news, because it confirmed that the physical thumb drive was appropriate, and the UEFI settings were correct. However, it lacked the drivers for basic things like keyboard. The display worked, and I was able to connect a keyboard.
## Third effort -- download and unpack drivers MSI
Downloaded the following file.

SurfaceLaptop7_ARM_Win11_26100_25.084.40393.0.msi

Expanded it on another computer, copied those drivers onto another thumb drive. Tried to make those drivers available to the booting machine, but it didn't work.
## Fourth effort -- add drivers to ISO
Tried many variations of following this set of instructions:

[Add Drivers to Windows ISO](https://woshub.com/integrate-drivers-to-windows-install-media/)

In the end, I tried adding the drivers to install.wim (index 3) and boot.wim (index 1 and 2). After an initial failure, I tried to scale back the set of drivers for the boot.wim, in case an unneeded driver was causing a problem.
“WinWork” is where all the work happens

Within that “Downloaded Resources contains the MSI with the drivers and the ISO. I already expanded the MSI into the Drivers folder. I took out some unnecessary drivers (audio and camera) and put that into the limited_drivers folder.

Start by mounting the ISO and copying from that to the ISO directory.

List the windows Images in install.wim
```
Get-WindowsImage -ImagePath C:\WinWork\ISO\sources\install.wim
```
Result:
```
ImageIndex       : 1
ImageName        : Windows 11 Home
ImageDescription : Windows 11 Home
ImageSize        : 20,237,673,068 bytes

ImageIndex       : 2
ImageName        : Windows 11 Home Single Language
ImageDescription : Windows 11 Home Single Language
ImageSize        : 20,219,225,142 bytes

ImageIndex       : 3
ImageName        : Windows 11 Pro
ImageDescription : Windows 11 Pro
ImageSize        : 20,523,301,778 bytes
```
I’m concerned with the Index 3, Windows 11 Pro image.

Mount the Windows 11 Pro Index 3 image in the mount directory:
```
Mount-WindowsImage -Path C:\WinWork\Mount\ -ImagePath C:\WinWork\ISO\sources\install.wim -Index 3
```
(Short wait for the above command to complete)

Add the drivers to the mounted image:
```
Add-WindowsDriver -Path C:\WinWork\Mount\ -Driver C:\WinWork\Drivers -Recurse
```
(Short wait for the above command to complete.)

Unmount the image:
```
Dismount-WindowsImage -Path C:\WinWork\Mount\ –Save
```
(Short wait for the above command to complete.)

While you’re waiting you can eject the mounted ISO.

When that is done mount the images in boot.wim onto mount2 and mount3, and add drivers to them.
```
Mount-WindowsImage -Path C:\WinWork\Mount2\ -ImagePath C:\WinWork\ISO\sources\boot.wim -Index 1
```
```
Add-WindowsDriver -Path C:\WinWork\Mount2\ -Driver C:\WinWork\limited_Drivers -Recurse
```
```
Dismount-WindowsImage -Path C:\WinWork\Mount2\ –Save
```

```
Mount-WindowsImage -Path C:\WinWork\Mount3\ -ImagePath C:\WinWork\ISO\sources\boot.wim -Index 2
```
```
Add-WindowsDriver -Path C:\WinWork\Mount3\ -Driver C:\WinWork\limited_Drivers -Recurse
```
```
Dismount-WindowsImage -Path C:\WinWork\Mount3\ –Save
```
Convert the WIM files to compressed images with integrated drivers (this was probably completely unnecessary):
```
DISM /Export-Image /SourceImageFile:C:\WinWork\ISO\sources\install.wim /SourceIndex:3 /DestinationImageFile:C:\WinWork\ISO\sources\install.esd /Compress:recovery
```
```
DISM /Export-Image /SourceImageFile:C:\WinWork\ISO\sources\boot.wim /SourceIndex:2 /DestinationImageFile:C:\WinWork\ISO\sources\boot.esd /Compress:recovery
```


This takes a long time.

Then write the ISO file:
```
.\oscdimg.exe -n -m -bc:\Winwork\iso\boot\etfsboot.com C:\winwork\ISO\ c:\NEW_SES_Win11_ISO_202509301355.iso
```

Then use Rufus to write it to the thumb drive.

This takes a long time.

Then an attempt to boot gave a ms updater error.
## Fifth effort -- Image directly from Microsoft
Microsoft updated their website to supply an image for the laptop that I was working on:

https://support.microsoft.com/en-us/surface-recovery-image

Downloading the provided image, and following the instructions to create a recovery drive yielded a drive from which the laptop could boot and install an OS.
Success.
