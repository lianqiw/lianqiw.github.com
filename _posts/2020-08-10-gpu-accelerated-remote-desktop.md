---
layout: post
title: GPU Accelerated Remote Desktop on Linux Server

---

Remote work has become the norm during the Covid-19 era. For us relying on high performance Linux servers, the need to connect to those servers remotely from home has become inevitable. There are multiple well established techniques that are fairly performant, such as

- [Turbovnc](https://www.turbovnc.org/)
- [Xrdp](http://xrdp.org/)
- [x2go](https://wiki.x2go.org/)

With [virtualGL](https://www.virtualgl.org/), it is even possible to run OpenGL accelerated 3D software remotely. However, none of the vnc or rdp based remote desktop currently support GPU acceleration in compressing the desktop image buffer for remote display. Graphics heavy application becomes terriblely slow for even very high performance servers. Try to play youtube videos remotely if you are not convinced. 

Luckly, there is a workaround solution based on [steam](https://store.steampowered.com/) that enables full GPU acceleration for both grabbing the display buffer and compressing for remote display. In order to have GPU acceleration, steam needs to run at DISPLAY=:0. Often there is no monitor attached to the server graphics card (e.g., Nvidia GPUs used for running CUDA software). Below, I will show you how to force the Xorg server to use a certain resolution. 

## Force resolution for disconnected display

The technique has been tested on a Centos 8 server with GTX980 GPU using Xorg and Xfce4. The tips are from [nvidia](https://http.download.nvidia.com/XFree86/Linux-x86/325.15/README/xconfigoptions.html) and the EDID bin files are downloaded from [edid-generator](https://github.com/akatrevorjay/edid-generator)

- Edit `/etc/X11/xorg.conf` Device section something like below

        Section "Device"
            Identifier     "Device0"
            Driver         "nvidia"
            BusID	   "PCI:2:0:0" #selects the GPU. DP-0 is output. Change if necessary
            Option	   "ConnectedMonitor" "DP-0" #Forces monitor
            Option	   "CustomEDID" "DP-0:/opt/cache/3840x2160.bin" #Forces mode
            Option	   "IgnoreEDIDChecksum" "DP-0"
            VendorName     "NVIDIA Corporation"
        EndSection

- If there is `/etc/X11/xorg.conf.d/10-nvidia.conf`, make sure it does not enable PrimaryGPU if it is not the GPU you want to use.

- Finally, you need to enable normal remote user to start xserver by editing (create if not exist) /etc/X11/Xwrapper.config with the following
      
        allowed_users=anybody
      
- now in any ssh session, startxfce and x11vnc. It is useful to start them in screen or tmux to avoid killing the X session when ssh is disconnected.

        startxfce4
        x11vnc -display :0 -rfbauth ~/.vnc/passwd

## Use steam for remote desktop

- In the remote session, install and start [steam](https://store.steampowered.com/) following the [instructions](https://access.redhat.com/discussions/4399951) here for Centos8. Login to your account. Go to Library, select `Add a game` from the lower left corner, and then select `add a non-steam game`. In the dialog, find and select `Mousepad` to add. 

- For desktop OS, such as Windows, Linux, or Mac OS, install the regular [steam](https://store.steampowered.com/). For iOS or Android, install the steam link app. Login to the same account. Goto Library page, `Mousepad` should appear in the game list. Click and stream it. If it doesn't work, try streaming some other application. 

- You will just see `Mousepad` in fullscreen. Click `Ctrl+Alt+D`, it will bring you to the desktop. Don't close `Mousepad` as it will disconnect the streaming. 

- Voila. You should now have a GPU accelerated remote desktop. Try to play youtube remotely to confirm the performance. You can safely turn off x11vnc now.
