Steam Gaming by Vulkan for Crostini, And This Might Be What Official "Steam for Chromeos(Aka Borealis)" Would Be like for My Chromebook

# Introduction
1 Years ago, mesa, the open-sourced implementation of OpenGL, Vulkan, and other graphics API specifications. Add experimental Vulkan support for virtio. Which means we can finally enable Vulkan support with GPU acceleration in Crostini, aka Linux Environment.  
## So What?
Almost all of Windows games support DirectX only. But it's Windows exclusive. So if Linux user want to use DirectX program, or play DirectX game, a translation layer is in need.  
The major solution is [DXVK](https://github.com/doitsujin/dxvk), A Vulkan-based translation layer for Direct3D 9/10/11 which allows running 3D applications on Linux using Wine.
However, **for a long time, ChromeOS Linux Environment aka Crostini didn't support Vulkan**. Sure you can rendering with CPU but it's so lag and not what I'm talking about here. I mean the Vulkan with GPU acceleration support.  
The most common way to play Steam game on Crostini is using Wined3d, a translation layer, translates DirectX to OpenGL, which is supported in the Crostini, to play Steam game. But there are often bugs and the performance is not ideal, compared to DXVK.   
**In short, Steam gaming is kind of broken on Crostini before.**

## Now?
So, with the experimental Vulkan support for virtio, we can finally exprience how DXVK would perform on ChromeOS Linux Environment.   
Most importantly, [Steam for ChromeOS(Borealis)](https://www.chromium.org/chromium-os/steam-on-chromeos/) is also run in container in VM, just like crostini, and also implemented using this driver.   
So, install and test the driver in Linux container now might be helpful to predict how Steam official support would perform on my Chromebook.    
**Even it's experimental, it can still run well. So if you are willing to test, It will be helpful to left your result in the comment. Especially for users with official Steam support. If you can do this test and make a comparison to the official Steam support.**
## About This Post
There're already many tutorials available. Those are really helpful. However, so matter the tutorial of installing arch on [Archlinux wiki](https://wiki.archlinux.org/title/Chrome_OS_devices/Crostini) or enable Vulkan support on [ChromeUnboxed's exclusive one](https://chromeunboxed.com/how-to-enable-vulkan-crostini/), are a little bit out-of-date. There are many differences to install and test it now. I encountered many pitfalls in the process of enabling and testing, so I'm sharing this note here to tell the difference and what you need to pay attention.
# Spec
Lenovo Chromebook Flex 5   
CPU: i5-10210U  
RAM: 8GB  
Storage: 256GB SSD.   
You can test on your devices too.
# Steps and Tips
I set all these things in **Stable channel**, if you're using **Beta or Dev(even Canary) channel**, there might be some difference, I will mention it below.
Remember this is only a note, not a full step-by-step tutorial, since there's so many tutorials available. But I would post the link of the tutorial I followed.
## Install Arch Linux
I basically following the tutorial on [archlinux wiki](https://wiki.archlinux.org/title/Chrome_OS_devices/Crostini). But there's some difference.  
### When You Have Multi-container Support
In **Dev and Canary channel**, multi-container is supported by default and it seems that you can't turn it off, and destroy termina might lead to a buggy situation, **so you don't need to "destroy termina" and rename container to penguin**, just simply using 
```
vmc container termina arch https://us.lxd.images.canonical.com/ archlinux/current
```
To install and configure as the wiki said. And click "arch" in Terminal to launch it.  
### Different Error Message
As the wiki, the error message after this step should be shown as:
```
"Error: routine at frontends/vmc.rs:403 `container_setup_user(vm_name,user_id_hash,container_name,username)` failed: timeout while waiting for signal"
```
However, there's still other error message something like:
```
no container found: arch
```
It's normal, run ```lxc list``` and you will find arch container is there, just continue configuring.
### Network Problem
If you really following the wiki step by step, you might facing the problem of no internet connection after reboot. **I meet this problem everytime in Dev channel almost everytime** The wiki gave you the [solution](https://wiki.archlinux.org/title/Chrome_OS_devices/Crostini#:~:text=should%20start%20normally.-,No%20network%20in%20container,-Note%3A%20In) **but when you saw this solution you've already lost internet connection and everything could be tough.** So, after runnng ```lxc exec arch -- bash``` and get into the arch shell, I recommend install ```dhclient``` and configure network immediately.
```
pacman -Syu
pacman -S dhclient
dhcpcd eth0
systemctl disable systemd-networkd
systemctl disable systemd-resolved
unlink /etc/resolv.conf
touch /etc/resolv.conf
systemctl enable dhclient@eth0
systemctl start dhclient@eth0
```
### Configure User
Everything should be fine following the [tutorial on wiki](https://wiki.archlinux.org/title/Chrome_OS_devices/Crostini#:~:text=exec%20arch%20--%20bash-,Set%20up%20the%20user,-The%20container%20creates)
### Configure Container
After installing [cros-container-guest-tools-git](https://aur.archlinux.org/packages/cros-container-guest-tools-git/), [wayland](https://archlinux.org/packages/?name=wayland) and [xorg-xwayland](https://archlinux.org/packages/?name=xorg-xwayland). **You need not only enable it, but start it.**
```
systemctl --user enable sommelier@0.service
systemctl --user start sommelier@0.service
```
So as ```sommelier@1.service``` ```sommelier-x@0.service``` ```sommelier-x@1.service```
### Replacing Container
Like what I mention at the beginning of this part. **When you have multi-container support, you don't really need to replace the default Debian Container.**
### Audio Issue
There might be some audio issue. And the solution is also provided on [Archlinux wiki](Audio%20playback/input). However in my cases I need to reinstall some software and manually start pulseaudio.    
```
pacman -S alsa-utils pulseaudio
cp -rT /etc/skel/.config/pulse ~/.config/pulse
pulseaudio --start
```
## Compile and Install Vulkan Driver
There're also some difference for me, compared to the tutorial on [ChromeUnboxed](https://chromeunboxed.com/how-to-enable-vulkan-crostini/) and [Luke Short's post](https://github.com/LukeShortCloud/rootpages/issues/495)
### Enable Vulkan When Start Termina
Make sure to restart termina with ```--enable-gpu``` and ```--enable-vulkan```.  
Press Ctrl+Alt+T to enter crosh.
```
vmc stop termina
vmc start --enable-gpu --enable-vulkan termina
```
###  Compiling and Install Vulkan Driver
Before we started, we need to turn on [multilib](https://wiki.archlinux.org/title/Official_repositories#multilib) support. 
Edit ```/etc/pacman.conf``` uncomment [multilib] section.
From
```
#[multilib]
#Include = /etc/pacman.d/mirrorlist
```
To
```
[multilib]
Include = /etc/pacman.d/mirrorlist
```
Then update system.
```
pacman -Syu
```
Then you can start to download, compile and install the latest mesa with experimental virtio Vulkan support. Everything is settled in PKGBUILD so what you don't have much things to do. Just simply install it.
```
git clone https://aur.archlinux.org/mesa-git.git
cd mesa-git
makepkg -si
```
This will take for a while, in my Chromebook it takes about 20 minutes to finish.
**However in my cases, the first time installing did't work properly. I have to install it again.**
```
makepkg -si
```
**The file is already compiled at the first time install, so the second time installation took only a few seconds.**
You still need to install the 32bit driver, or your Steam game via DXVK might not launch.
```
git clone https://aur.archlinux.org/lib32-mesa-git.git
cd lib32-mesa-git
makepkg -si
```
**Also, like the mesa-git, I installed it twice.**
```
makepkg -si
```
Like the ```mesa-git```, The first time it took about 20 minutes to compile, and the second time only took a few seconds.
## Check If It works
You need ```vulkan-tools``` to see if it works.
```
sudo pacman -S vulkan-tools
```
**However, when I type ```vulkaninfo```, I get this error code.**
```
ERROR at /build/vulkan-tools/src/Vulkan-Tools-1.3.213/vulkaninfo/vulkaninfo.h:231:vkGetPhysicalDeviceSurfaceFormats2KHR failed with ERROR_SURFACE_LOST_KHR
```
But ```vkcube``` works fine. I don't know what's that error mean, if someone can explain it. It would help a lot.  
But since ```vkcube``` works fine, finally, let's try Steam games.
## Steam Games
Finally, let's test Steam Games using DXVK
### Install Steam
Steam is available in [multilib](https://wiki.archlinux.org/title/Official_repositories#multilib) and we've enable multilib before. So just simply install it.
```
sudo pacman -S steam
```
And type ```steam``` in the terminal, press enter to launch it. You can also launch it from your ChromeOS app launcher. However running in terminal can see the steam output, if something goes wrong we can see the error code in terminal immediately. Which is good for fixing it.   
For example, sometimes you might see
```
fail to load steamui.so
```
And you need to remove the .steam folder in home directory.
```
cd ~
rm -rf .steam
```
Then start Steam, it will fix itself.
### Proton
When You login Steam, you need to enable Steam Play for all game. This will use Proton, a translation layer to play Windows game, with DXVK built-in.
**Open Steam -- Settings -- Steam Play, check "Enable Steam Play for all other title".** You can choose what Proton version you want. I left "Proton Experimental" by default.   
Press OK, restart Steam, and now you can install the game you want to test, just simply hit install like what you're doing on your regular PC.  
### Performance
In theory, the game compatibility should be the same as [protondb](https://www.protondb.com/) tells you.   
But many popular game in my library is officially support Linux, like **Portal**. So I would like to test some games that are not so popular, to see if DXVK can work properly.
#### Unconnected Marketeers
![th18](https://i.ibb.co/TL4vZss/Screenshot-2022-05-15-02-53-14.png)
I've test ***Unconnected Marketeers***, an arcade style 2D STG game, which is marked PLATIUM on SteamDB. With ```DXVK_HUD=devinfo,fps %command%```, shows I'm using DXVK, venus drive, run at smoothly 60 fps.   
What about 3D titles?  
#### Deemo -Reborn-
![th18](https://i.ibb.co/bKTBNmc/Screenshot-2022-05-15-14-27-41.png)
I try another title ***Deemo -Reborn-*** . It's a adventure rhythm game with beautiful 3D graphic, and you can't adjust the graphic setting except for resolution. In wined3d, this game will runs in weird graphic glitch. But in Vulkan, everything works fine. And after setting it to low resolution, I got about 15-20FPS. Actually I'm surprised to see this game runs at this frame rate. Remember this is an experimental drive and the game runs in a container in VM at Intel UHD GPU. So If we have other 3D game, which can adjust the graphic, maybe lower graphics may save the frame rate.
## But It's bad, Isn't It?
Yes, it's not as good as I expect. However, maybe the official Steam support will be another story.   
Even use the same VM, same container, same drive, official Steam support also provided Game Mode, to let the game run as smoothly as possible.   
Last but for all, if you can join the testing, it would be really helpful.
