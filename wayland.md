# Wayland

**outdated - updated version will come soon once everything is in ports!**
## Introduction
Current Wayland support is highly experiemental and a work in progress. 
Use these instructions if you are interested in trying Wayland out. 
Testers and contributers are very welcome!

Pull requests for improvements to this document are also welcome.

## Prerequisites 

### Software
Build and install world and kernel from  
Base repository: [github.com/freebsddesktop/freebsd-base-graphics](https://github.com/freebsddesktop/freebsd-base-graphics "github")  
Branch: *drm-next*

Build and install ports from  
Ports repository: [github.com/freebsddesktop/freebsd-ports-graphics](https://github.com/freebsddesktop/freebsd-ports-graphics "github")  
Branch: *xserver-mesa-next-udev*

Wayland related ports are located in the `$PORTSDIR/wayland` directory for now. Default options for ports should be good.

### Hardware
Wayland is confirmed to work on  

#### Intel (Haswell or older) 
By enabling evdev in kernel config and [patching](https://reviews.freebsd.org/D7496 "link") drm for kevent the stock FreeBSD 12 source code can be used. 

#### Intel (any version)
Use base from repository above for any Intel hardware.

#### Radeon 
[needs confirmation]

#### Nvidia 
[needs confirmation]

#### VirtualBox
Weston now runs in VirtualBox (or any EFI-enabled machine without GPU driver) thanks to new SCFB backend (requires vt, aka newcons).


## Configuration

### Evdev
As can be seen in [sys/dev/evdev/evdev.h](https://github.com/FreeBSDDesktop/freebsd-base-graphics/blob/drm-next-4.7/sys/dev/evdev/evdev.h#L48-L58)
add to /etc/sysctl.conf to enable/disable input devices. For example for only hw devices,  
`kern.evdev.rcpt_mask=12`

### Weston
Default weston.ini was modified with path to Xwayland, background pattern image and a launcher icon for weston-terminal. 
The icon might be installed by the xfce4 desktop environment.
```
[core]
modules=xwayland.so

[shell]
background-image=/usr/local/share/weston/pattern.png

[launcher]
icon=/usr/local/share/icons/Adwaita/24x24/apps/utilities-terminal.png
path=/usr/local/bin/weston-terminal

[xwayland]
path=/usr/local/bin/Xwayland
```

### User & Groups
DRM belongs to the video group so make sure to add yourself.  
`$ sudo pw groupmod video -m $USER`

A group *input* is created for input devices, add yourself to that too.  
`$ sudo pw groupmod input -m $USER`  

Add a devfs rule for evdev devices.  
```
# /etc/devfs.rules
[localrules=10]
add path 'input/*' mode 0660 group input 
```
And load these in rc.conf  
```
# /etc/rc.conf
devfs_system_ruleset="localrules"
```
## Running

### Weston
Package install will create the *weston-launch* group. Add yourself to that group.  
`$ sudo pw groupmod weston-launch -m $USER`  
Use *ctrl+alt+backspace* to exit.  
VT switching works with *ctrl+alt+Fx*.  

#### DRM (Intel)
When logged in to console make sure to load drivers  
`$ sudo kldload i915kms`  (if you're on Intel)
and start weston with  
`$ weston-launch`

#### SCFB (hardware independent - requires UEFI and VT)
The SCFB backend can be used by starting with  
`$ weston-launch -- --backend=scfb-backend.so`  
Weston will be started in the current virtual console. 
Debug output will overwrite the graphics so you might want to redirect to /dev/null or a log file like:  
`$ weston-launch -- --backend=scfb-backend.so >& weston.log`  

The SCFB backend is fairly untested and might have bugs. 


Weston also installs many example programs, just type   
`$ ls -l /usr/local/bin/weston*`  
to see what is available. 

### Sway (or other wlc-based compositor)
Start with  
`$ sway`

Current only supports x11 and drm backend, the latter requires a hardware graphics driver.

## Known bugs
#### WLC
VT/TTY switching does not work with wlc-based compositors. Input device can not be re-opened on resume for some reason. WIP to find the reason for this.

#### Intel - Cherryview
Graphics gets distorted randomly when for example running weston-terminal in Weston. Correct rendering can be achieved by resizing the window to certain sizes. 

## Other

All these packages are work in progress and contain a lot of debug output and experimental features.  
Crashes might happen, please don't use on a system that contains valuable data.  
