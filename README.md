# Installing on VNC Server on Solaris 11.31 #

### 1. Check then system 

```sh
# uname -a
SunOS SOL 5.11 11.3 sun4v sparc SUNW,Sun-Fire-T200

# vncserver
-ksh: vncserver: not found [No such file or directory]
```

### 2. Install VNC

```sh
# pkg install solaris-desktop  
Creating Plan (Checking for conflicting actions): |
            Packages to install: 156
            Services to change:  11
        Create boot environment:  No
Create backup boot environment: Yes

DOWNLOAD                                PKGS         FILES    XFER (MB)   SPEED
Completed                            156/156   16131/16131  214.9/214.9    0B/s

PHASE                                          ITEMS
Installing new actions                   28121/28121
Updating package state database                 Done 
Updating package cache                           0/0 
Updating image state                            Done 
Creating fast lookup database                   Done 
Reading search index                            Done 
Building new search index                  1075/1075 
Updating package cache                           2/2 
```

### 3. Set Password

```sh
# vncserver

You will require a password to access your desktops.

Password:
Verify:
xauth:  file /root/.Xauthority does not exist

New 'SOL:1 ()' desktop is SOL:1

Creating default startup script /root/.vnc/xstartup
Starting applications specified in /root/.vnc/xstartup
Log file is /root/.vnc/SOL:1.log
```

### 4. Change Default Window Desktop Manager (twn -> gnome)

Kill the session

```sh
# vncserver -kill :1
Killing Xvnc process ID 28044
```

Edit Script 
Change 'twm &' -> 'gnome-session &'

```sh
# vi /root/.vnc/xstartup

#!/bin/sh

vncconfig -iconic &
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
OS=`uname -s`
if [ $OS = 'Linux' ]; then
case "$WINDOWMANAGER" in
    *gnome*)
    if [ -e /etc/SuSE-release ]; then
        PATH=$PATH:/opt/gnome/bin
        export PATH
    fi
    ;;
esac
fi
if [ -x /etc/X11/xinit/xinitrc ]; then
exec /etc/X11/xinit/xinitrc
fi
if [ -f /etc/X11/xinit/xinitrc ]; then
exec sh /etc/X11/xinit/xinitrc
fi
if [ -x /etc/gdm/Xsession ] && [ -x /usr/bin/dtstart ] ; then
exec /etc/gdm/Xsession /usr/bin/dtstart jds
fi
if [ -x /etc/X11/gdm/Xsession ] && [ -x /usr/bin/dtstart ] ; then
exec /etc/X11/gdm/Xsession /usr/bin/dtstart jds
fi
if [ -x /usr/dt/config/Xsession.jds ]; then
exec /usr/dt/config/Xsession.jds
fi
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey
xterm -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
#twm &
gnome-session &
```


Restart VNC Server

```sh
# vncserver

New 'SOL:1 ()' desktop is SOL:1

Creating default startup script /root/.vnc/xstartup
Starting applications specified in /root/.vnc/xstartup
Log file is /root/.vnc/SOL:1.log
```

### 5. Connect to Server


### 6-1. If you can't connect, then check xvnc-inetd service

```sh
# inetadm | grep xvnc-inetd
disabled  disabled       svc:/application/x11/xvnc-inetd:default

# svcadm -v enable xvnc-inetd   
svc:/application/x11/xvnc-inetd:default enabled.

# svcadm -v restart xvnc-inetd:default                                                                                                   
Action restart set for svc:/application/x11/xvnc-inetd:default.

root@vitec-sun:/etc/gdm# inetadm | grep xvnc-inetd                                                                                                              
enabled   online         svc:/application/x11/xvnc-inetd:default

root@vitec-sun:/etc/gdm# vncserver
```

### 6-2. Allow to guest connect 

```sh
# vi /etc/gdm/custom.conf 
[xdmcp]
Enable=true

# svcadm -v restart gdm
```

