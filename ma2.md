# How to transfer savefiles from demo?

Your savefile from demo can be moved to full released game. Full game will automatically convert the savefile to match the new format when you try to open the savefile from demo.

Follow platform specific instructions:

 * N64 Console
For EverDrive X7 your savefile is in folder "gamedata". Rename "ma2d.srm" to "ma2.srm" to match the name of the ROM you are playing. For SummerCart the savefile is near Z64 so make sure that savefile has the correct name.

 * Parallel Launcher
Click on "ma2d" ROM and select option "Show Save File". Do the same for "ma2" ROM you either downloaded from RHDC directly or from other source. Rename the savefile for "ma2d" ROM to match "ma2" ROM. Make sure you are using the latest version of Parallel Core (look at guide below)

 * Luna Project64
Works out of box but make sure that you are using v3.6.1 or higher.

 * RMG
RMG is no longer supported. Please use Parallel Launcher. To import the savefile right click on "ma2" and select "Import Project64 savefile". Go to "Save" folder near RMG.exe and select file "MARIO ADVENTURE 2-_______.sra". Your savefile should be imported. Note that this guide doesn't work till falcobuster fixes bug in Parallel Launcher save importing.

 * Luna Mupen64PlusAE on Android
Download APK from releases again (version 1.5). It should automatically transfer the savefile but I am not 100% sure it will work. Sorry if it does not. 

# Supported platforms:

 * N64 Console + SummerCart/EverDrive X7
    Culling features are used to ensure high performance which can lead to occasional flickering. You are _highly_ recommended to use carts that support autosaving.
    Consider buying SummerCart64 here https://www.aliexpress.com/item/1005007816403187.html?gatewayAdapt=usa2glo4itemAdapt, recommended by MVG https://www.youtube.com/watch?v=K1yQ5l-DnQc

 * Parallel Launcher with Parallel core v2.21.2 or higher + OGRE https://parallel-launcher.ca/
    Check for updates with Settings - Updaters - Check for Updates

 * Luna Project64 v3.6.1 or higher with ANGLE GLideN64 4.3.21 or higher https://github.com/Luna-Project64/Luna-Project64/releases
    Options - Configuration - Plugins - Video plugin - ANGLE GLideN64
    No FB emulation (Options - Graphics Settings - Frame buffer - uncheck Emulate frame buffer) 
    Enable Noise (Options - Graphics Settings - Emulation - check Enable Noise)

 * Android: Luna Mupen64PlusAE https://github.com/aglab2/mupen64plus-ae/releases
    This worked on my phone but I expect support to be sporadic. Best use emulators above on PC.
