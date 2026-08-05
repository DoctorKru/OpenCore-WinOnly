# OpenCore-WinOnly
Minimal configuration of OpenCore 1.0.7 bootloader to boot the only copy of Windows 10/11. The purpose is to tweak mainboard firmware on modern HP/Dell laptops and apply custom ACPI overrides if needed.

Based on: https://github.com/acidanthera/OpenCorePkg/releases/tag/1.0.7

From Reddit: 
> *Many modern Dell and HP laptops ship with Intel GPUs that fully support HEVC (H.265) in hardware — yet Windows reports HEVC as unsupported. Installing the Microsoft HEVC codec does nothing. Drivers are current. Linux works. Windows doesn’t. This is not a driver bug. This is intentional firmware gating, implemented via ACPI tables..* 

### I found, that you can enable HEVC hardware support without ACPI table override, at least on HP laptops, thanks to: 

https://github.com/acidanthera/OpenCorePkg

Just boot Windows via OpenCore bootloader and you're good to go! OpenCore send fake firmware configuration to Windows and breaks the logic of HEVC resrtriction bit, which should be applied on HP hardware only. You need Secure Boot Off and Fast Startup Off, but Testsign mode not needed! The solution tested on Win10/11 on certain HP laptops. So:

1. Download minimal "Win boot only" Opencore configuration from here or create your own config with OpenCorePkg

2. At first, put EFI folder to bootable USB stick (use Rufus etc.) and boot PC from USB (be sure, you do not have any acpitabl.dat in Windows\System32\ and hiberfil.sys in the root).

3. Run Msinfo32 in Windows and check strings in System Summary: System Manufacturer, Model, Baseboard Manufacturer, Bios Version etc. - they should show something like "Acidanthera", "iMac19,1" etc. The same check you can do by CPU-Z -> Mainboard tab. The lack of any HP indication means, that you have successfully booted Windows via OpenCore.

4. Now, check HEVC decoding capabilities, i recommend DXVA Checker https://bluesky-soft.com/en/DXVAChecker.html as it shows HEVC profiles on the first screen.

5. Finally check HEVC HW decoding with 4K-8K hevc video file, higher FPS also recommended. You should get smooth playback with near zero CPU load and up to 30% GPU on modern laptops. To force HW HEVC in VLC: Preferences → All, Input / Codecs → Video codecs → FFmpeg → Hardware decoding → Direct3D11

6. No HEVC HW decoding even after successful Windows via OpenCore boot. <br> Then, you need custom ACPI override. You can do it directly in OpenCore, which was developed by Hackintosh people who are experts in ACPI tables patching and have done everything possible to make the process more convenient. Dump ACPI tables directly in OpenCore, search for restriction bit with Intel ACPICA tools, compile your custom overrides and put "ready to go" .aml files in Opencore ACPI folder. 

7. If all OK, then you can inject Opencore to your SSD primary EFI partition to make it permanent bootloader for Windows or leave it on USB stick as "HEVC hardware dongle" for rare use.

### Injection OpenCore to your SSD primary EFI partition:
- To be safe, backup EFI partition with Acronis or similar program
- Mount EFI partition of your main SSD in Windows. Run CMD as Admin:
```cmd        
diskpart
    list disk
    select disk 0   ← usually your main SSD
    list vol
    select vol 3 ← the 100-500MB FAT32 volume labeled "System" or "EFI"
    assign letter=Z
    exit
```
 or single command line, if you know disk and vol numbers:
```cmd        
(echo select disk 0 && echo select volume 3 && echo assign letter=Z) | diskpart
```
- Run file manager as Admin (Total Commander etc.) to edit EFI partition content
- Put BOOTx64.efi from OpenCore-WinOnly/EFI/BOOT to your SSD../EFI/BOOT. If any BOOTx64.efi already exists in SSD../EFI/BOOT rename it.
- Put OC folder from OpenCore-WinOnly/EFI to your SSD../EFI
- Rename config_SSD.plist to config.plist in OC folder. You can also delete .contentFlavour and .contentVisibility from /EFI and EFI/BOOT.
- Setup Boot Manager in Windows. Run CMD as Admin:
```cmd        
bcdedit /set {bootmgr} path \EFI\BOOT\BOOTx64.efi
```
```cmd        
bcdedit /set {fwbootmgr} displayorder {bootmgr} /addfirst
```
- Reboot
> [!NOTE]
> The "3 reboots" effect can occure: at 1st boot you get pure Windows, at 2d boot you get Windows via Opencore and 3d boot additionally creates HP "Windows" entry in OC picker which you should ignore and press Down key to load Windows + OpenCore. The same problem can occure on very cold boot. HP firmware sometimes recreates default boot paths if you occasionally entered HP BIOS settings, even without saving changes or just open F9 boot menu. 

> [!TIP]
> To clear NVRAM in OC picker when needed press ***Space → Clear NVram***.
- Useful checks, helping to get working configuration:
  ```cmd        
    bcdedit /enum {bootmgr} ← Windows Boot Manager configuration
    bcdedit /enum firmware ← Shows every UEFI entry your BIOS sees
    bcdedit /enum {current} ← Shows what Windows is running on right now.
  ```
  In OC Picker menu press Space → OpenShell.efi
  ```js        
    Shell> bcfg dump -v ← show boot options
  ```
