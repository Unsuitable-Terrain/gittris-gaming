# Mint 22 - System install instructions for WINE and Lutris gaming

I prefer Debian based distros - Debian, Mint. I appreciate it isn't always bleeding edge, but it doesn't really need to be.

My main focus, though, is always stable driver support. A lot of work goes into distros, but to replace a Windows gaming system, I find Mint has Linux up and running very quickly. A different distro might work for you, but always make sure you've got the right graphics drivers and Vulkan support.

I choose to shun snap and flatpak for their random issues and added complications they can bring. I'm a "professional" user, who buys legitimate products, and always has an eye on security so I can do this with confidence. Your experience with such tools may be flawless, or you may be paranoid about security. 

# Away we go!

## Add 32bit support 
This is for WINE and other apps that need it
```
sudo dpkg --add-architecture i386
```
## GPU Drivers
### NVIDIA
Install latest [NVIDIA drivers from their website](https://www.nvidia.com/en-us/drivers/) - This should also pull in vulkan support. 

Your distro might have already staged this. For Mint22, pick a version for your GPU. For very new GPUs use 565 onwards. For my middle-of-the-road GPU:
```
sudo apt install nvidia-driver-550 -y
```

### AMD
Stick with repo Mesa drivers (reasonably up to date now.) Alternatively, this PPA is well maintained: [Kisak's PPA](https://launchpad.net/~kisak/+archive/ubuntu/kisak-mesa) which is reliable

Mesa and vulkan install command for ubuntu24/mint22
```
sudo apt install libegl-mesa0 libegl-mesa0:i386 libegl1-mesa-dev libgl1-mesa-dri libgl1-mesa-dri:i386 libglapi-mesa libglapi-mesa:i386 libglu1-mesa libglu1-mesa:i386 libglu1-mesa-dev libglx-mesa0 libglx-mesa0:i386 libosmesa6 libosmesa6:i386 mesa-utils mesa-utils-bin mesa-va-drivers mesa-va-drivers:i386 mesa-vdpau-drivers mesa-vdpau-drivers:i386 mesa-vulkan-drivers mesa-vulkan-drivers:i386 libvulkan1 libvulkan1:i386
```

*Restarting the system at this point is a good idea!*

# Option1: Lutris flatpak
Not my choice, but I thought I'd include it as the easy option for some. Install flatpak, install Steam, install Lutris. Deal with random sandbox issues :P

## flatpaks (not ikea)
Note that some distros do not add flathub as a source. You'll have to do that manually. Mint, however, does. When you install Steam flatpak, you want the app that says "app/com.valvesoftware.Steam/x86_64/stable" 
```
sudo apt install flatpak -y
flatpak install com.valvesoftware.Steam
flatpak install net.lutris.Lutris
```

Run Steam and login.
Run Lutris and wait for it to download its support files. It should list "Steam" as a source on the left, and because you logged in, your games library should appear.

# Option2: Lutris Git (preferred)
So you're sticking with the Git version? Good for you. You need to install Git, WINE pre-reqs, a couple of tools to support the scripting system and Steam.

## Pre-req1: WINE staging from WineHQ

Install instructions for WINE for Ubuntu 24 (noble) and Mint 22 see [WineHQ.org](https://www.winehq.org) for others
```
sudo mkdir -pm755 /etc/apt/keyrings
sudo wget -O /etc/apt/keyrings/winehq-archive.key https://dl.winehq.org/wine-builds/winehq.key
sudo wget -NP /etc/apt/sources.list.d/ https://dl.winehq.org/wine-builds/ubuntu/dists/noble/winehq-noble.sources
sudo apt update && sudo apt install --install-recommends winehq-staging -y
```

## Pre-req2: Steam and tools
Install the Steam installer, basic tools, lutris Python and script pre-reqs
```
sudo apt install steam-installer curl cmake git build-essential scdoc python3-pil python3-numpy python3-build python3-hatchling python3-installer python3-yaml python3-venv cabextract 7-zip unrar -y
```

Run Steam once so it sets up and patches, and then login. As an alternative to Steam Proton and occasionally better performance in certain games, also grab the latest [GE proton](https://github.com/GloriousEggroll/proton-ge-custom), untar to ~/.steam/root/compatibilitytools.d/ (needs to be created)

## Optional
Lutris can also use vulkan-tools (provides the "vulkaninfo" command) and fluidsynth, but the source commands may depend on your distribution. For Ubuntu 24/Mint22 however
```
sudo apt install vulkan-tools fluidsynth
```

## "Install" Lutris Git
I prefer the git version. I've already explained why in the first couple of paragraphs.

## Git Lutris and Git umu-launcher
Now handled by the handy "gittris" script in this very repo!

### Automagic, assuming you trust my script
```
mkdir ~/source && cd ~/source && git clone https://github.com/jmczab/git-tris
~/source/git-tris/gittris
```

### Manual, if you don't trust my script :)
Create a directory for your git clone. Pull in 
```
mkdir ~/mygitstuff && cd ~/mygitstuff && git clone https://github.com/lutris/lutris.git
```
Link and run Lutris once, let it install stuff and update
```
sudo ln -s ~/mygitstuff/lutris/bin/lutris /usr/local/bin/lutris
lutris -d
```

Install Git umu-launcher so Lutris can access and use Steam protons - preferred over the static version, as updating git keeps them in step
```
cd ~/mygitstuff && git clone https://github.com/Open-Wine-Components/umu-launcher.git
cd ~/mygitstuff/umu-launcher && ./configure.sh --user-install && make 2>&1 >>/dev/null && make install
```

### Create a desktop launch item
Simple really. Here's an inline script. Change the Lutris icon path as needed.
```
IFS="~~~"
read -r -d '' lutrisDesktop << EOF 
[Desktop Entry]
Encoding=UTF-8
Version=1.0
Type=Application
Exec=/home/userName/.local/bin/lutris
Name=Lutris
Comment=Lutris game platform
Terminal=false
Icon=/home/userName/source/lutris/share/icons/hicolor/scalable/apps/lutris.svg
PrefersNonDefaultGPU=false
EOF
echo ${lutrisDesktop} | sed -e s/userName/`whoami`/g > ~/.local/share/applications/lutris.desktop
```

### Add "lutris:" links in browser
Process will vary per browser, but as a guide, click a Lutris install link from the site, and change it in "applications" or "file types".

# Ready player tux
System is pretty much ready to install games, and game!


## Optional steps
### For Ubuntu
I would fire snapd into the heart of the sun - because I hate the interface to snap, I'd rather sandbox my own stuff when required, and FlatPak is much more robust
```
sudo apt autoremove --purge snapd
```
### Games in a tidy place
I prefer to create a separate location for games, and use the builtin games group for games access (setgid: not for everyone I realise, but it helps permissions) also add "myuser" to the input group for access to certain devices, like dance mats. Yes, I still enjoy [Stepmania](https://github.com/stepmania/stepmania)
```
sudo usermod -a -G games myuser
sudo mkdir /opt/games
sudo chown myuser:games /opt/games
sudo chmod g+ws /opt/games
sudo usermod -a -G input myuser 
```
You might need to restart before this works

Create a separate SteamLibrary location for steam to store its games, and one for Lutris
```
mkdir /opt/games/SteamLibrary
mkdir /opt/games/lutris
```

Add /opt/games/SteamLibrary to Steam's default library location, and update Lutris preferences to use /opt/games/lutris

Obvs if you are skipping and didn't follow that you need to subsitute "myuser" for your actual user name, then slow down a bit :)
