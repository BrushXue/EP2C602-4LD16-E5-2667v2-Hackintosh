# ASRock  EP2C602-4L/D16 E5-2670 Hackintosh Guide
Still working on the Chinese version.

This guide works on macOS Catalina (10.15.4). C602 is even more complicated than X79 so I don't recommend beginners try this build.

I referenced the [guide](https://www.tonymacx86.com/threads/guide-asrock-rack-ep2c602.289060/) from teamawesome. This repo is the supplementary of my own setup.

![image](Screenshot_en-us.png)

## Hardware
| Item | Brand | Model | Driver | Comment |
|-----|-----|-----|-----|-----|
| Motherboard | ASRock | EP2C602-4L/D16 | | |
| CPU | Intel | Dual Xeon E5-2670 | | |
| RAM | Samsung | 16x8GB RECC DDR3L 1333 | | |
| dGPU | AMD | FirePro W5100 4GB | built-in | FakeID 0x665C1002 |
| SSD | Crucial | MX200 500GB | | |
| Ethernet | Intel | 82574L | built-in | Flash Subvendor ID|
| Monitor | Dell | U2718Q | | |

## Hardware Preparation
### Flash BIOS
Unlike ASRock X79s, C602 motherboards have CFG locks. And for some reason, the `AppleIntelCPUPM` patch in Clover won't work for Sandy Bridges CPUs. So you have to flash the BIOS to unlock it. I'm providing my modified BIOS file in `Utilities/P602_4D1.90`. If you have other similar motherboard, simply use [UEFIPatch](https://github.com/LongSoft/UEFITool/releases) to patch it.
### Flash NIC
The motherboard has Intel 82574L which is natively supported. However the different subvendor ID prevents the system from loading the kext. So you need to use Linux to flash the EEPROM. (I'm using centOS as an example)
```
su root
yum install -y ethtool
ifconfig
```
Try to find out which port you are using. In my case, it's `enp9s0`. Then run the following command:
```
ethtool -E enp9s0 magic 0x10D38086 offset 0x16 value 0x00
ethtool -E enp9s0 magic 0x10D38086 offset 0x17 value 0x00
ethtool -E enp9s0 magic 0x10D38086 offset 0x1A value 0xF6
ethtool -E enp9s0 magic 0x10D38086 offset 0x16 value 0x00
ethtool -E enp9s0 magic 0x10D38086 offset 0x17 value 0x00
ethtool -E enp9s0 magic 0x10D38086 offset 0x18 value 0x86
ethtool -E enp9s0 magic 0x10D38086 offset 0x19 value 0x80
```
## BIOS Setup
| Name | Option |
| --- | --- |
| VT-d | Disabled |
| Marvell 9230 | Disabled |
| SCU devices | Disabled |
| Serial Port 1 | Disabled |
| Serial Port 2 | Disabled |

* These options must be disabled. There're more options which you can play around. teamawesome provided a page-by-page guide to setup BIOS.
## Post Installation
I assume that you are not beginnners and know how to install Hackintosh. So I omitted all trivial steps.

This ASRock motherboard doesn't have `bcfg` command in UEFI shell. So you need first boot from USB and use the UEFI shell in Clover to add the boot menu. Assume that your EFI partition is `fs0`, run the following commands:
```
fs0:
cd \efi\boot
bcfg boot add 0 bootx64.efi "Clover"
reset
```
Now you should be able to boot from SSD.

I have customized my USB Port files. However if your motherboard is different from mine, you may want to use `USBInjectAll.kext` and do that again by yourself.

You are all set.

## Comments
1. `/EFI/CLOVER/ACPI/patched/SSDT-1.aml` renames GFX0 to GFX1, which is neccesary to inject GPU fakeID on SMBIOS MacPro6,1. If your GPU is natively supoorted, you can remove this SSDT file and disable **ATI Fake ID**/**Inject ATI** in Colver.
2. This FirePro GPU doesn't support HiDPI output natively. To enable that, use this [script](https://github.com/xzhih/one-key-hidpi).
## Known issue
I could't find a solution to get power management running on **dual Sandy Bridge** system. So I'm simply using `NullCPUPowerManagement.kext` to reduce power consumption. If you figured out how to deal with the CPUPM, please let me know!
## Acknowledgement
- Apple for the macOS
- teamawesome for the initial idea of the guide
- CloverHackyColor for CloverBootloader
- Acidanthera for Lilu, Whatevergreen and VirtualSMC
- naveenkrdy for AppleMCEReporterDisabler
- corpnewt for NullCPUPowerManagement
- mackie100 for Clover Configurator
- headkaze for Hackintool
