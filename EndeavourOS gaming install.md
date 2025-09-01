# Perilous Endeavour(O)S
Getting Git Lutris and games running quickly on EndeavourOS didn't prove to be too challenging for someone with a Fedora/Debian Linux (and HP-UX, AIX, Solaris, but that knowledge is very old) background. That said, there are quirks.

## Choice of deskop
I picked Cinnamon. I like cinnamon. It is lightweight, pretty tidy, works with Wayland, and works with Lutris. Plasma is nice, sure, but has way to many features included and enabled by default. I spent a lot of time turning off accessibility options I didn't choose, and a bunch of defaults that interfere with games.

## Driver installation

### Nvidia
After much browsing of OS notes and self determination - Install the nvidia install tool, then run it to install nvidia drivers over nouveau
```
sudo pacman -S nvidia-inst
sudo nvidia-inst
```



### AMD/Mesa 
Mostly on by default, but adding 32 bit support helps for those legacy issues
Pick one of vulkan-radeon+lib32-vulkan-radeon (open source) OR amdvlk+lib32-amdvlk (A port of AMD's own)
`sudo pacman -S lib32-mesa vkd3d lib32-vkd3d vulkan-mesa-layers lib32-vulkan-mesa-layers lib32-vulkan-icd-loader vulkan-radeon lib32-vulkan-radeon`

## Give me Steam
```
pacman -S steam
```
If you chose nvidia, it wants a version of lib32-vulkan-driver, so choose the one aligned with your driver choice, in my case lib32-nvidia-utils. Run and sign into Steam at least once.

## Why must you always WINE, Louis?
Staging is always a better bet IMO. The repo is a bit behind winehq builds, but take we can get, because I don't want to be building it right now.
`pacman -S wine-staging`

## Lutris support packages
Basic stuff really - stuff Lutris (and winetricks) needs
`sudo pacman -S git 7zip vulkan-tools cabextract innoextract curl`

## Python packages
After some poking - Arch didn't go for python/python3 foolery, so its just a case of determining the equivalent package names
`sudo pacman -S python-semantic-version python-watchdog python-requests python-numpy python-build python-hatchling python-installer python-yaml python-virtualenv python-pip python-uv python-lxml python-certifi`

## Lutris install
Just cloned it from git, as per my usual M.O, Linked the main script into my user's ~/.local/bin and created a desktop entry
```
cat > ~/Desktop/Lutris.desktop << EOF
[Desktop Entry]
Name=Lutris
Exec=/home/user/source/lutris/bin/lutris
Comment=
Terminal=false
PrefersNonDefaultGPU=false
Icon=cinnamon-panel-launcher
Type=Application
EOF
```

## Lutris integration
Lutris normally fixes up the clickable Firefox link integration for lutris: links, but it was broken. Had to manually put it in my profile's handlers.json, if you need it.

## Game installation failures!
*1. vulkan failure using wine (causes error 256)*
I tried installing the following using pacman: -
- vkd3d lib32-vkd3d - yes! :D

*2. Error 256 after game install*
This was due to winetricks requiring cabextract. Normally I install it, but forgot. It's now in the pre-reqs above.

*3. Game failed to launch with a dxvk error.*
Had to work out the packages required...oh look, we're missing 32 bit.
`pacman -S lib32-nvidia-utils`

# Game on
That was it really. A couple of hours and I now know EndeavourOS is a reasonable platform to do Lutris in a hurry.

# Odds and sods
`sudo pacman -S meson ninja cmake tree dosbox stb glm libdisplay-info system-config-printer doomretro libcurl-gnutls lib32-libcurl-gnutls`
