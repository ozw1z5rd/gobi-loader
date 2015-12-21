# Introduction #

Another gobi loader ( not for the 2k ) version which work for the HP EliteBook 6930p.

# Details #

Unless you switch the notebook off and then switch it on, the firmware is not unloaded from the device. If you are going to modify the loader it's a good idea to reset the device before each load.

This loader works with "ubuntu 2.6.32-30-generic" or major.

Be sure the wifi support is on before to load the linux kernel. The wifi led must be "Blue", if it does not the kernel will not see the gobi modem.

If anything is ok the mesg output must show:
```
 [    8.504631] USB Serial support registered for Qualcomm USB modem
 [    8.505413] qcserial 1-2:1.0: Qualcomm USB modem converter detected
 [    8.505527] usb 1-2: Qualcomm USB modem converter now attached to ttyUSB0
```

If this does not happen add hp-wmi and qcserial to /etc/modules.
This is how looks my /etc/modules file:

```
$ cat /etc/modules 
# /etc/modules: kernel modules to load at boot time.
#
# This file contains the names of kernel modules that should be loaded
# at boot time, one per line. Lines beginning with "#" are ignored.

lp
rtc
hp-wmi
qcserial
```

If you change the /etc/modules, please reboot.

# Get the firmware #
The Gobi firmare are in **C:/QUALCOMM/QDLService/Packages** folder in the windows partition, if you left it, otherwise get a windows machine and download the driver from the HP site.

As you can see there are plenty of firmware.
I'm using these:

```
md5=3c43ae175b5a74a066b901d4dbb7b933  amss.mbn
md5=19480cce0c24ba5766f70d0faec909fd  apps.mbn
```

and I moved these in /lib/firmware/gobi

Ok. Now you can start anything...

```
gcc gobiLoader.c -o gobiLoader
./gobiLoader /dev/ttyUSB0 /lib/firmware/gobi


Starting Gobi firmware loader 
QDL protocol server request sent (QDL download)
QDL protocol server response received (QDL download)
QDL protocol server request sent (QDL download )
QDL protocol server response received (bad?wait?)
Device ok. Sending amss.mbn
QDL protocol server request sent (Send AMSS)
QDL protocol server response received (Send AMSS)
QDL protocol server request sent (Complete AMSS)
..........
AMSS Sent.
QDL protocol server response received (Send AMSS)
Sending apss.mbn
QDL protocol server request sent (Send Apps)
QDL protocol server response received (Send Apps)
QDL protocol server request sent (Complete Apps)
....
QDL protocol server response received (Complete Apps)

Apps Sent.
Finishing...
QDL protocol server request sent (Complete QDL download)
Waiting 2sec to allow the device startup..

```

Ok, now you got it working.
If the load process stop after the QDL download request is sent, the modem has already a firmware loaded. Switch the notebook off and try again. Sometime the load process completes and the modem is not yet initialized, in this case try again to load it.

Don't use **gnome-ppp**, I noticed some trouble when sending the strings to the modem, use wvdial which works fine.