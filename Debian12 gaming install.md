# Debian 12 - System install instructions for WINE and Lutris gaming

Difference on the Mint install instructions, but for Debian 12 with a focus on Nvidia

# Away we go!

## Add 32bit support 
No, we're not done with 32 bit yet. This is for older WINE games and other apps that need it.
```
sudo dpkg --add-architecture i386
```

## GPU Drivers
### NVIDIA
Install latest [NVIDIA drivers from their website](https://www.nvidia.com/en-us/drivers/) - This should also pull in vulkan support. 

Install the .deb from Nvidia, and install the gpg key for the repository (a message will appear if you run apt update)
Then apt-update, and install the driver. 

*Note: the PATH of the cp command will change with driver versions - this is the version you get as at Jan 2025*
```
sudo dpkg -i nvidia.deb
sudo cp /var/nvidia-driver-local-repo-debian12-550.54.14/nvidia-driver-local-64642CF5-keyring.gpg /usr/share/keyrings/
sudo apt update
sudo apt install nvidia-driver -y
```

*Info: If it fails to install as it is missing "nivida-cleanup" or similar, then as I said in the first install step: **enable the "non-free" packages via the software and applications tool***

### AMD
#### Mesa and vulkan for Debian 12
Well, Mesa for Deb12 was, for a long time, stuck on 22.
However, yay for backports - we now have [Mesa 24](https://packages.debian.org/source/bookworm-backports/mesa)

This should cover more of your AMD needs and give you better functionality. Still not great for DX12 though.

```
sudo echo "deb http://deb.debian.org/debian bookworm-backports main" >>/etc/apt/sources.list.d/debian-12-backports.list
sudo apt update
sudo apt install mesa-vulkan-drivers
```

*Restarting the system at this point is a good idea!*

# Lutris Debian 12 pre-reqs
First, enable the "non-free" packages via the software and applications tool - otherwise some pre-reqs do not get picked up. It is also required for the nvidia driver. Then run apt update. Also note that if you used the minimalist ISO image (which is aimed at servers rather than desktops) you will not have certain commands by default, or a wide selection of desktops. Still, install some basic tools first:-
```
sudo apt update
sudo apt install wget curl 7zip git
```

## Full install or flatpak? 
If you are going to use flatpak for everything, then skip these three pre-reqs

### 1. Python and supporting tools
This is not a complete list to support all Lutris installers, but covers most of it.

```
sudo apt install innoextract unrar-free vulkan-tools fluidsynth fluid-soundfont-gs cabextract
sudo apt install cmake build-essential scdoc python3-magic python3-lxml python3-pil python3-numpy python3-build python3-hatchling python3-installer python3-yaml python3-venv -y
```
### 2. WINE staging from WineHQ
Install instructions for WINE for Debian 12 (bookworm) see [WineHQ.org](https://www.winehq.org) for others
```
sudo mkdir -pm755 /etc/apt/keyrings
wget -O - https://dl.winehq.org/wine-builds/winehq.key | sudo gpg --dearmor -o /etc/apt/keyrings/winehq-archive.key -
sudo wget -NP /etc/apt/sources.list.d/ https://dl.winehq.org/wine-builds/debian/dists/bookworm/winehq-bookworm.sources
sudo apt update && sudo apt install --install-recommends winehq-staging
```

### 3. Steam
You can download the packaged steam installer, or download the generic deb [Steam's website](https://steampowered.com) and install.
```
sudo apt install steam-installer
```
Find the steam-installer in the menu and run it, it will install Steam.
Once Steam runs and patches, login at least once.

# Lutris choices

# Option1: Lutris flatpak
Not my choice, but I thought I'd include it as the easy option for some. Install flatpak, install Steam, install Lutris. Deal with random sandbox issues :P

## flatpaks (not ikea)
Note that some distros do not add flathub as a source. You'll have to do that manually. The Steam flatpak, is the app that says "app/com.valvesoftware.Steam/x86_64/stable" 
```
sudo apt install flatpak -y
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak install com.valvesoftware.Steam
flatpak install net.lutris.Lutris
```

Run Steam and login
Run Lutris and wait for it to download its support files. It should list "Steam" as a source on the left, and because you logged in, your games library should appear.

**Note:** If you go with flatpak, you will likely need to install other flatpak apps as well to support Lutris' full emulation suite (e.g. dolphin for wii/switch) but on the bright side, you can skip some pre-reqs. However, having supporting pre-reqs installed will give you a safe fallback.

## Option2: Lutris package

A debian compatible and up-to date lutris package is built by strycore on Opensuse's site. See Lutris.net's downloads for details and if you prefer a manual download instead. Alternatively, download the .deb from the GitHub releases page.

```
echo 'deb http://download.opensuse.org/repositories/home:/strycore/Debian_12/ /' | sudo tee /etc/apt/sources.list.d/home:strycore.list
curl -fsSL https://download.opensuse.org/repositories/home:strycore/Debian_12/Release.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/home_strycore.gpg > /dev/null
sudo apt update
sudo apt install lutris
```

Then continue on to pre-reqs

## Option3: Lutris Git (preferred)
So you're sticking with the Git version? Good for you. You need to install Git, WINE pre-reqs, a couple of tools to support the scripting system and Steam.

### What you need: Git Lutris and Git umu-launcher
Can install them yourself, or use the handy "gittris" script in this very repo!

### Automagic git install, assuming you trust my script
```
mkdir ~/source && cd ~/source && git clone https://github.com/Unsuitable-Terrain/gittris-gaming
~/source/git-tris/gittris
```

### Manual install, if you don't trust my script :)
TL;DR: Create a directory for your git clone. Pull Lutris, then link in to somewhere in your PATH
```
mkdir ~/mygitstuff && cd ~/mygitstuff && git clone https://github.com/lutris/lutris.git
sudo ln -s ~/mygitstuff/lutris/bin/lutris /usr/local/bin/lutris
```
Install Git umu-launcher so Lutris can access and use Steam protons - which are much preferred over the automatic versions, as updating keeps them in step
```
cd ~/mygitstuff && git clone https://github.com/Open-Wine-Components/umu-launcher.git
cd ~/mygitstuff/umu-launcher && ./configure.sh --user-install && make 2>&1 >>/dev/null && make install
```

### Create a Lutris desktop launch item for Git install
Simple really. Here's an inline script that assumes the main Lutris script has been linked into to your .local/bin
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

### Add "lutris:" link support in browser
Process will vary per browser, but as a guide, click a Lutris install link from the site, and then change it in "applications" or "file types" to launch with Lutris.

## Ready player tux
System is pretty much ready to install games! Lutris can link to your GoG, then directly download and install available games. It also runs the BattleNet and Epic game platform clients nicely.

# Optional steps

## GloriousEgroll's (no, not making that up) custom Proton
As an alternative to Steam Proton and occasionally better performance in certain games, also grab the latest [GE proton](https://github.com/GloriousEggroll/proton-ge-custom), extract to ~/.steam/root/compatibilitytools.d/ (may need to be created). It is something Lutris also provides, just slightly lagging behind. Just remember you will have to maintain it.

## Run Lutris, update winetricks
Lutris downloads a version of winetricks, but obvs there are updates. For the Lutris version
```
~/.local/share/lutris/runtimes/winetricks/winetricks --self-update
```

For flatpak winetricks
```
~/.var/app/.local/share/lutris/runtimes/winetricks/winetricks --self-update
```

## Games in a tidy place
I prefer to create a separate location for games, and use the builtin games group for games access (setgid: not for everyone I realise, but it helps permissions) also add "myuser" to the input group for access to certain devices, like dance mats. Yes, I still enjoy [Stepmania](https://github.com/stepmania/stepmania)
```
sudo usermod -a -G games myuser
sudo mkdir /opt/games
sudo chown myuser:games /opt/games
sudo chmod g+ws /opt/games
sudo usermod -a -G input myuser 
```
You need to restart before this works

My preference is to split a SteamLibrary location for Steam to store its games, and one for Lutris
```
mkdir /opt/games/SteamLibrary /opt/games/lutris
```

Add /opt/games/SteamLibrary to Steam's default library location, and update Lutris preferences to use /opt/games/lutris

Obvs if you are skipping and didn't follow that you need to subsitute "myuser" for your actual user name, then slow down a bit :)

That about covers the basics. Lutris is ready - just use Runner integration to download and install things from GoG, Epic etc.
Steam is installed and will use its own Proton to run games, so your compatibility will be "anything that runs on steam deck" and more!


# Final note
Debian 12 defaults to an older version of the Linux 6 kernel, so will not support bleeding edge hardware.
You'll need to grab an updated kernel from a reliable source, or build it yourself (for fun and profit.)
