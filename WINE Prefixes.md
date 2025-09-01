TOC
1. [What is WINE?](#whatiswine)
    1. [Where to get WINE and what to install?](#wheretogetwine)
    2. [What does Proton have to do with WINE?](#protonvswine)
    3. [Why do you capitalise WINE?](#wthdude)
2. [What is a WINE prefix and how is it used?](#wineprefixes)
    1. [How do I create a WINE prefix?](#creatingprefixes)
    2. [How do I interact with an existing WINE prefix?](#usingprefixes)
    3. [What is winetricks and how do I use it with WINE?](#whatiswinetricks)
    4. [What are the most common WINE variables, and what is multislot-wine?](#variablesandmultislot)
    5. [How do I use a specific version of WINE with a WINE prefix, and what about non-multislot?](#specificwineversions)
    6. [What are the limitations on architecture?](#winearch)
    7. [How do I install common Windows apps into a WINE prefix?](#commonapps)
    8. [What is a DLL override and why do I need it?](#dlloverides)
    9. [I am getting a prompt for something called mono/gecko?](#monoandgecko)
    10. [My Windows application does not support my Linux native language](#winelanguage)
3. [Examples](#examples)
    1. [create new wineprefix, then run and installer you downloaded](#exp1)
    2. [create new wineprefix then install .NET 4.8 using winetricks and windows core fonts - quietly and unattended](#exp2)
    3. [create new 32 bit wineprefix](#exp3)
    4. [I'm running 4k screen and the Windows application fonts are TINY](#exp4)


<div id='whatiswine' />

# *What is WINE?*
WINE is a set of compatibility tools that allows Windows commands and operating system calls to work on Linux, mainly by mapping them to their equivalent in Linux.
The upshot of which is it allows you to run a lot of Windows software on Linux.

<div id='wheretogetwine' />

## *Where to get WINE and what to install?*
The main Linux distributions provide their own WINE via their software repositories, but check against WINEHQ if they are kept up to date.

My personal choice for pre-packaged WINE on Debian is the source - [winehq.org](https://www.winehq.org)

The instructions are on the Downloads page for your flavour of Linux - enable their repository as a source, then update sources and install.

I enable 32-bit support, and use wine-staging which is "in development," but not bleeding edge. 
Using the recommended winehq-staging is enough to support everything I need.

However, you may not have a wide choice. If you have doubts, search your software repo for wine. If it still isn't clear, ask on the forums for your distro.

<div id='protonvswine' />

## *What does Proton have to do with WINE?*
Proton is a version of WINE packaged and built to provide more focussed support for Windows games. A bunch of developers got together (mainly Valve, but also many notable others) produce and maintain it, or forks of it (thank you, GloriousEggroll.) Things like the Steam deck rely on it to run Windows titles.

<div id='wthdude' />

## *Why do you capitalise 'WINE'?*
Well, it is an acronym for "wine is not emulation" or "wine is not an emulator" (which would be WINAE and just ...doesn't work)

It is "not an emulator" because it maps functions, rather than providing code (emulation) to do it, but there is always a line for emulation that gets broken somewhere :)

<div id='wineprefixes' />
  
# *What is a WINE prefix and how is it used?*
In short? A prefix is just a directory with supporting files in it, so it looks like a Windows OS.
- **dosdevices** - symlinks mimicing drives available to the prefix
- **drive_c** - Obviously containing the C: drive contents
- **various reg files** - mimicing the registry and storing Windows settings

<div id='creatingprefixes' />

## *How do I create a WINE prefix?*
The most basic way is to create a new directory, set the variable WINEPREFIX to that directory, then execute any wine command (which will trigger a wineboot to prepare the prefix if it hasn't been done before.)

For example, this will create a new prefix
```
mkdir $HOME/mynewprefix
export WINEPREFIX=$HOME/mynewprefix
wine wineboot
```
**Note:** I use "export WINEPREFIX" so every wine command I issue in that terminal window after that uses the same prefix.
You can also prefix the prefix to every wine command... *WINEPREFIX=$HOME/mynewprefix wine wineboot* ...which is better for automated builds and scripts.

<div id='usingprefixes' />

## *How do I interact with an existing WINE prefix?*
Again, set the WINEPREIX variable to its directory, so any WINE-related commands after that will be executed in that prefix.
So if you did...
```
export WINEPREFIX=/home/user/myprefix
wine winecfg
```
...it would run the wine configuration panel in that prefix using the system WINE.

<div id='whatiswinetricks' />
Winetricks is a scripting tool that allows you to install common features into WINE prefixes without much hassle. Because it is a script, it can be used to customise or build prefixes with ease.
It has a UI and all you need to do is set WINEPREFIX, run winetricks, select the prefix, and you have a bunch of common options that allow you to install things or change your prefix.

For example, to install .NET4.8 in a WINEPREFIX you'd need to manually do download and install it.
Winetricks does that for you - and a lot more besides. Here's a short list of things: -
- dotNet 4+
- DXVK for gaming
- VisualC Runtimes
- DLL workarounds

There is a usually a verison of it in your distribution's repository, and although typically those are out of date very fast, the script can be told to --self-update and will pull the latest verison from GitHub. It contains sources and checksums for many popular Microsoft downloads, so needs a check every few months or so.


<div id='variablesandmultislot' />

## *What are the most common WINE variables, and what is multislot-wine?*
WINE uses few variables - see 'man wine' for a full list and the specific version info below, but thanks to multislot WINE, you'll only need two: -
- WINEPREFIX - where to store windows files - set this to the directory of the prefix (i.e. the directory that contains dosdevices, drive_c etc.)
- WINEARCH - architecture - set to win32 if you need 32 bit, will default to win64

These days, WINE is typically built as "multislot" - meaning the wine executable knows where to find its associated files, based on its local path. So if you invoke a specific wine command, it will pick up the relevant wineserver, mono, libaries etc. This allows you to have multiple versions of WINE installed on a system, and easily choose which one to run without much hassle.


<div id='specificwineversions' />

## *How do I use a specific version of WINE with a WINE prefix, and what about non-multislot?*
If you have multiple versions of WINE, and every time you interact with a prefix, the version of WINE you run will reconfigure it to its "expected" settings. 

If you have all multislot WINE installs, this isn't much of an issue - just invoke the wine you want against the prefix.
```
WINEPREFIX=~/myprefix /opt/wineversions/wine8/bin/wine explorer
```

However, you may also have a WINE version that **isn't multislot** and doesn't play well, so will need to set *all* (or most) of the variables before running WINE commands.
```
export WINEVERPATH="/opt/my-custom-wine"
export PATH="$WINEVERPATH/bin:$PATH"
export WINESERVER="$WINEVERPATH/bin/wineserver"
export WINELOADER="$WINEVERPATH/bin/wine"
export WINEDLLPATH="$WINEVERPATH/lib/wine/fakedlls"
export LD_LIBRARY_PATH="$WINEVERPATH/lib64:$WINEVERPATH/lib:$LD_LIBRARY_PATH"
export WINEARCH=win32
export WINEPREFIX=/home/user/mycustomprefix
$WINELOADER $WINEPREFIX/drive_c/Apps/Appname/software.exe
```
<div id='winearch' />

## *What are the limitations on architecture?*
The best thing is to focus on is compatibility, but try and select the highest possible.
A 64 bit prefix will only run 64 and 32 bit code. A 32 bit prefix will run 32 bit and 16 bit code. See WINEARCH variables above!

For example, you might know of some 32 bit software, and decide to put it in a 64 bit prefix. However, the software comes with a launcher or service that is 16 bit, so a 32 bit prefix is the only option. Another example is  supporting software might only come as a 16 bit installer.

<div id='commonapps' />

## *How do I install common Windows apps into a WINE prefix?*
To be honest, the best and quickest way is to use [winetricks](https://github.com/Winetricks/winetricks)
- Winetricks allows you to easily and quickly install DLLs, common software (like .NET, or Visual C Runtime, the all-important DXVK) fonts and settings (like windows version) into any prefix.
- Winetricks is a shell script, but can run in the user context.
- Winetricks caches downloads for reuse (~/.cache/winetricks) so disk use can grow a bit without you realising!

<div id='dlloverides' />

## *What is a DLL override and why do I need it?*
WINE provides replacement functions for a LOT of Windows DLLs, but some software needs functions only present in the original file. The use of these is determined by the DLL overrides. If you run the configuration tool (wine winecfg) you can see the overrides under the "Libraries" tab. 

Each entry corresponds to a DLL, with various options of native/builtin. This choice just means "use the disk DLL"/"use the wine DLL" when the DLL function is called.
These can be set on the command line, by the configuration tool, or by winetricks.

<div id='monoandgecko' />

## *I am getting a prompt for something called mono/gecko?*
### Mono
If you installed WINE with "recommends" it should have also have pulled a package called "mono-runtime" or similar.

Mono is a lightweight replacement for .NET in WINE prefixes. Installing it in a new prefix does not cause any issues. The prompt is usually a new version and is updating its runtimes, or detecting a requirement.

However, some games dependant on .NET run into issues, so you can replace mono with a full fat .NET install using winetricks. This will automatically remove mono, if installed.

### Gecko
Gecko is another compatibility layer - Because Windows integrates internet exporer with its OS, WINE uses its own layer. Gecko is an extended compatibility layer, but is rarely needed for games.

<div id='winelanguage' />

## *My Windows application does not support my Linux native language*
Old Windows software typically relies on the system language being some variant of US English. However, if you don't use English and have a non-default keyboard, this can be a problem. Generally, the variables are stored in the registry under HKEY_CURRENT_USER\Control Panel\International

You can set the LANG environment variable before launching WINE, which will cause WINE to rewrite some of the Windows system variables. 
```
mkdir ~/winelanguages
export WINEPREFIX=~/winelangages
LANG=en_US.UTF-8 wine wineboot
LANG=en_US.UTF-8 wine regedit
```

**Note:** Language settings will get reset every time you run a WINE command in this prefix without the same LANG set.

You can make changes in the registry - like the default date formats, again, using the same LANG setting so it persists, then use regedit to view them, then launch the exe.
```
LANG=en_US.UTF-8 wine reg add "HKCU\Control Panel\International" /v sLongDate /t REG_SZ /d "ddd, MMM d, yyyy" /f
LANG=en_US.UTF-8 wine regedit
LANG=en_US.UTF-8 wine $WINEPREFIX/drive_c/MySoftware/application.exe
```

<div id='examples' />

# Examples

<div id='exp1' />

### Example 1 - create new wineprefix, then run and installer you downloaded
```
mkdir -p /opt/wine/newsoftware
export WINEPREFIX=/opt/wine/newsoftware
wine wineboot
wine ~/Downloads/software_installer.exe
```

<div id='exp2' />

### Example 2 - create new wineprefix then install .NET 4.8 using winetricks and windows core fonts - quietly and unattended
```
mkdir -p /opt/wine/appneedingdotnet48
export WINEPREFIX=/opt/wine/appneedingdotnet48
winetricks -q dotnet48 corefonts
```
If you have a mono-runtime installed, this will automatically execute remove_mono so there is no conflict.

<div id='exp3' />

### Example 3 - create new 32 bit wineprefix
```
mkdir -p /opt/wine/wine32bit
export WINEPREFIX=/opt/wine/wine32bit
export WINEARCH=win32
wine wineboot
```
When interacting with this prefix in future, WINE will know it is 32 bit and behave accordingly.

<div id='exp4' />

### Example 4 - I'm running 4k screen and the Windows application fonts are TINY
```
export WINEPREFIX=/opt/wine/myfontsaretiny
wine winecfg
```
No, no Windows dynamic DPI here :) Go into the "graphics" tab and move the DPI slider - I find 144 or 168 acceptable.
