# Dell-7010-rebar-guide
How to guide aimed at helping others get Dell 7010s into gaming shape


Sources:
https://www.tachytelic.net/2021/12/dell-optiplex-7010-pcie-nvme/ 


    1. Things you’ll need before you start:
        1. The latest driver .exe file
            1. Go to the site and download the latest (A29) https://www.tachytelic.net/2021/12/dell-optiplex-7010-pcie-nvme/ 
        2. A flash drive with freedos
            1. follow this guide: https://support.onlogic.com/documentation/dosusb/?_gl=1*161ulsp*_ga*MTQwOTQ4NjIyNC4xNzE4NTkxMDk0*_ga_SEVJD5HQBB*MTcxODU5MTA5NC4xLjAuMTcxODU5MTA5NS42MC4wLjA.*_gcl_au*MTcwODUxOTUwNy4xNzE4NTkxMDk1 
            2. Move the driver exe file over to this flash drive. I placed mine at the base of the drive
        3. A flash drive with win 10 or 11 installer
            1. Go to the installation media website and get the version you want to work with in the future. However which version won’t matter for the purpose of this tutorial: https://support.microsoft.com/en-us/windows/create-installation-media-for-windows-99a58364-8c02-206f-aa6f-40c3b507420d 
            2. Once you’ve created a installer flash, go into its efi/boot/ folder. Add ‘.old’ to the boox64.efi file
            3. download modgrubshell.efi from https://github.com/datasone/grub-mod-setup_var/releases/ 
            4. put it in the efi/boot/ folder and rename it to match the original file’s original name.
        4. Install Intel Management Engine: https://dl.dell.com/FOLDER04469185M/5/Intel-Management-Engine-Components-Installer_C3VMM_WIN_11.0.6.1194_A02.EXE 
        5. Download Intel System Tools: https://www.tachytelic.net/wp-content/uploads/Intel-ME-System-Tools-v8-r3.zip 
        6. Download the 0.28 UEFI Tool version: https://github.com/LongSoft/UEFITool/releases/download/0.28.0/UEFITool_0.28.0_win32.zip 
        7. Download MMTool https://groups.google.com/g/microsoft.public.win32.programmer.kernel/c/SS1arKGD_Pg?pli=1 
        8. Download the 7010 specific files https://drive.google.com/drive/folders/1bKjuearyBexTY4RDtL_OqfhRuf_sQQ-Y?usp=sharing
        9. Download ReBarDxe.ffs and ReBarState.exe from https://github.com/xCuri0/ReBarUEFI/releases

Ok, so what are each of these things and why are we getting them? 
We’re grabbing the latest driver because in the case of the 7010, we know it works with this methond. Also, this process has to start from an existing driver, so might as well get the latest and greatest. We’re getting the exe file because dell doesn’t distribute their bios files on their own. If you use someone elses bios file instead of one created on your own machine from running the exe, you’ll find its incompatible. This is because dell bakes in the device info into the bios file, making it so the exe’s update method is the only way to get a bios that works on your machine. 

Flash drive with free dos is required in order to run the ‘.exe’ file. You’ll find the bios installer often times will not work from windows. I’m unsure why this is, but running it from dos seems to get around that limitation. We’ll be booting from dos and then running the exe file

flash drive with win 10 or 11 is required to get grub working. Basically the Grub tool works by replacing the efi file in a standard windows install media drive. Fun fact, if you boot from the drive in legacy mode, it’ll function as normal because it won’t leverage the .efi file. I try to keep the original efi file so I can revert the drive back to being a working installer later if I want.

Intel Management Engine is needed to be able to extract the bios. The Intel System Tools contains the bios flashing tools you’ll need to extract your current bios, and then to flash on your modified bios later
the weird stuff comes from needing both the UEFI Tool and MMTool. The UEFI tool is required for inserting the new ffs file with the rebar functionality into your bios. However, you’ll also be updating existing entries. If you try to do this with UEFI Tool, it will fail to work. MMTool on the other had can do the replacements without corrupting the bios.

The 7010 files are files you’d normally have to create via patching. However, unlike the bios itself, these are reusable as long as your working across 7010 motherboards (so don’t reuse them on an ASUS motherboard)

    2. Start with an updated Bios.
        1. Plug in your dos drive
        2. Reboot the computer and hit F12 to bring up the boot menu
        3. select your dos drive, boot into dos
        4. run the installer (if you type O7 and then hit tab, it should autofill the whole file name, and you can then hit enter). Yes your way through the options.It will require a reboot, and you’ll want to either hit F2 for the bios or hit F12 and select the bios option
    3. We now need to setup the bios. Flashing the bios wipes all the settings you may have had prior to this point
        1. Disable Legacy Boot options
        2. Set your bios to use uefi boot
        3. Make sure secure boot is off
        4. disable C State
        5. Change any other settings your machine needs, such as the boot order you want. You basically need every bios setting set before the next step

Ok, so what’re we doing here? We’re creating a solid bios to build on. Running the bios installer gets us on the latest, and then we’re making sure the bios settings match what we’ll need by the end. The biggest thing is when we wrap this process up, you can’t have Legacy or Secure Boot going, or after these mods, the bios won’t display video. 

    4. Set the service mode jumper: https://www.tachytelic.net/2021/12/dell-optiplex-7010-pcie-nvme/
        1. either snag the password reset jumper, or put something conductive between the two pins as it boots. It’ll retain this setting until you do a hard shut down 
    5. Open the command prompt as an admin
    6. navigate to C:\Users\YourAccount\Downloads\Intel ME System Tools v8 r3\Flash Programming Tool\Windows64. 
    7. Backup the bios with fptw64 using ‘fptw64.exe -d backup.bin’
    8. Open the backup.bin with the uefi tool
    9. Find the pcibus entry with the search. Write click it and use insert after and insert the ReBarDxe.ffs file (this would also be the opportune time to do the nvme mod if you want to do that as well, but what ffs file you need and where to put it is in the tachytelic.net guide.)
    10. Save as a modified.bin file. Next thing would normally be to do the dsdt and patching... But I am giving you the patched files.
    11. With MMTool, open the modified.bin file 
    12. Replace each entry that matches what I attached, namely:
        1. AmiBoardInfo.ffs
        2. PchS3Peim.ffs
        3. SciBus.ffs
        4. PCIHostBridge.ffs
        5. Runtime.ffs
    13. Save out a final.bin file 
    14. Apply it with ‘fptw64 -bios -f final.bin’
    15. WARNING, this next step requires the bios setting to be correct, or you’ll brick your bios! Restart and boot into a flash drive with grub installed on it.
    16. From grub use ‘setup_var 0x2 0x1’
    17. Reboot
    18. Run the the ReBarState.exe program
    19. Check GPU-Z, go to advanced, select resizable bar in the drop down, and everything should be good

    
