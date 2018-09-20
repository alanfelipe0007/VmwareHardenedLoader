# VmwareHardenedLoader
Vmware Hardened VM detection mitigation loader

For now, only Windows (vista~win10) x64 guests are supported.

It get vmware guest undetected by VMProtect 3.2 (anti-vm feature).

## What it does

the VmLoader driver patches SystemFirmwareTable at runtime, it removes all detectable signatures like "VMware" "Virtual" "VMWARE".

## Warning

Do not install vmtools, it will ruin everything!

use TeamViewer / AnyDesk / mstsc / VNC viewer instead!

## 1st Step: Add following settings into .vmx

```
hypervisor.cpuid.v0 = "FALSE"
board-id.reflectHost = "TRUE"
hw.model.reflectHost = "TRUE"
serialNumber.reflectHost = "TRUE"
smbios.reflectHost = "TRUE"
isolation.tools.getPtrLocation.disable = "TRUE"
isolation.tools.setPtrLocation.disable = "TRUE"
isolation.tools.setVersion.disable = "TRUE"
isolation.tools.getVersion.disable = "TRUE"
monitor_control.disable_directexec = "TRUE"
monitor_control.disable_chksimd = "TRUE"
monitor_control.disable_ntreloc = "TRUE"
monitor_control.disable_selfmod = "TRUE"
monitor_control.disable_reloc = "TRUE"
monitor_control.disable_btinout = "TRUE"
monitor_control.disable_btmemspace = "TRUE"
monitor_control.disable_btpriv = "TRUE"
monitor_control.disable_btseg = "TRUE"
```

## 2nd Step: Modify MAC address

Modify guest's MAC address to whatever except below:
```
	TCHAR *szMac[][2] = {
		{ _T("\x00\x05\x69"), _T("00:05:69") }, // VMWare, Inc.
		{ _T("\x00\x0C\x29"), _T("00:0c:29") }, // VMWare, Inc.
		{ _T("\x00\x1C\x14"), _T("00:1C:14") }, // VMWare, Inc.
		{ _T("\x00\x50\x56"), _T("00:50:56") },	// VMWare, Inc.
	};
```

![mac](https://github.com/hzqst/VmwareHardenedLoader/raw/master/img/4.png)

I use 

## 3rd Step: Load vmloader.sys in vm guest
open command prompt as System Administrator, use the following commands

```
sc create vmloader binPath= "\??\c:\vmloader.sys" type= "kernel"
sc start vmloader
```

If an error occurs when start service, use DbgView to see whats happened. you can post an issue with DbgView output information if it does't work on your environment.

If no error, then everything works fine.

you could put "vmloader.sys" wherever you want, except vmware shared folders.

when you no longer need the mitigation, use
```
sc stop vmloader
sc delete vmloader
```
to unload the driver.

## Vmware guest win8.1 x64 with VMProtect 3.2 packed program (anti-vm option enabled)

![before](https://github.com/hzqst/VmwareHardenedLoader/raw/master/img/1.png)
![sigs](https://github.com/hzqst/VmwareHardenedLoader/raw/master/img/2.png)
![after](https://github.com/hzqst/VmwareHardenedLoader/raw/master/img/3.png)

## License
This software is released under the MIT License, see LICENSE.

Some util procedures are from https://github.com/tandasat/HyperPlatform

https://github.com/aquynh/capstone is used to disasm ntoskrnl code.
