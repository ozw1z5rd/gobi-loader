</p>
<h1>Ubuntu 11.04 miss the support for my notebook</h1>
<p>
I installed ubuntu 11.04 and nothing is working.
</p>
<p>
The modem is not detected at all!
</p>
<p>
As first step add the following two rows into the /etc/modules files:
</p>
<p><pre><code>
hp-wmi
qcserial
</code></pre></p>
<p>
This is still not enough because the vendorID / deviceID used in my notebook are missing into the qcserial.ko driver.
</p>
<p>
I mean these numbers:
</p>
<p><pre><code>
 oz@ubuntu:/usr/src/linux-source-2.6.38/linux-source-2.6.38/drivers/usb/serial$ lsusb
 :
 .
 Bus 003 Device 002: ID 03f0:171d Hewlett-Packard Wireless (Bluetooth + WLAN)  Interface [Integrated Module]
 -----------------------^^^^^^^^^
 :
 .
</code></pre></p>
<p>
Let's add them and recompile qcserial.c:
</p>
<p><pre><code>
 apt-get install linux-source-2.6.38
 cd /usr/src/linux-source-2.6.38
 bunzip2 linux-source-2.6.38.tar.bz2
 tar xvf  linux-source-2.6.38.tar
 cd  linux-source-2.6.38/drivers/usb/serial
 vi qcserial.c
</code></pre></p>
<p>
Add this row :
</p>
<p><pre><code>
       {USB_DEVICE(0x03f0, 0x171d)},   /<em> HP Gobi 2000 Modem device OZ  </em>/
</code></pre></p>
<p>
Open the MakeFile inside the same folder and add in top:
</p>
<p><pre><code>
 obj-m = qcserial.o
 KVERSION = $(shell uname -r)
 all:
 &lt;press tab&gt;make -C /lib/modules/$(KVERSION)/build M=$(PWD) modules
 clean:
 &lt;press tab&gt;make -C /lib/modules/$(KVERSION)/build M=$(PWD) clean
</code></pre></p>
<p>
Now
</p>
<p><pre><code>
 make
</code></pre></p>
<p>
after the compilation completes copy qcserial.ko in
</p>
<p>
/lib/modules/2.6.38-8-generic/kernel/drivers/usb/serial/
</p>
<p>
After the reboot you will see the modem is detected and the ttyUSB0, ttyUSB1 and ttyUSB2 are made available:
</p>
<p><pre><code>
root@ubuntu:/root# uname -a
Linux ubuntu 2.6.38-8-generic #42-Ubuntu SMP Mon Apr 11 03:31:24 UTC 2011 x86_64 x86_64 x86_64 GNU/Linux
root@ubuntu:/root# sh loadfm.sh
loading firmware
Starting Gobi firmware loader
QDL protocol server request sent (QDL download)
QDL protocol server response received (QDL download)
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
....QDL protocol server response received (Complete Apps)
Apps Sent.
Finishing...
QDL protocol server request sent (Complete QDL download)
Waiting 2sec to allow the device startup...
dmesg
[   66.143267] qcserial 1-2:1.1: Qualcomm USB modem converter detected
[   66.143355] usb 1-2: Qualcomm USB modem converter now attached to ttyUSB0
[   66.144809] qcserial 1-2:1.2: Qualcomm USB modem converter detected
[   66.144900] usb 1-2: Qualcomm USB modem converter now attached to ttyUSB1
[   66.145698] qcserial 1-2:1.3: Qualcomm USB modem converter detected
[   66.145767] usb 1-2: Qualcomm USB modem converter now attached to ttyUSB2
Network manager handles it correctly.
Jul 12 07:15:10 ubuntu modem-manager[860]: &lt;info&gt;  (ttyUSB0) opening serial port...
Jul 12 07:15:10 ubuntu modem-manager[860]: &lt;info&gt;  (ttyUSB1) opening serial port...
Jul 12 07:15:10 ubuntu modem-manager[860]: &lt;info&gt;  (ttyUSB2) opening serial port...
Jul 12 07:15:22 ubuntu modem-manager[860]: &lt;info&gt;  (ttyUSB1) closing serial port...
Jul 12 07:15:22 ubuntu modem-manager[860]: &lt;info&gt;  (ttyUSB1) serial port closed
Jul 12 07:15:25 ubuntu modem-manager[860]: &lt;info&gt;  (ttyUSB1) opening serial port...
Jul 12 07:15:25 ubuntu modem-manager[860]: &lt;info&gt;  (Gobi): GSM modem /sys/devices/pci0000:00/0000:00:1a.7/usb1/1-2 claimed port ttyUSB1
Jul 12 07:15:46 ubuntu modem-manager[860]: &lt;info&gt;  (ttyUSB0) closing serial port...
Jul 12 07:15:46 ubuntu modem-manager[860]: &lt;info&gt;  (ttyUSB0) serial port closed
Jul 12 07:15:46 ubuntu modem-manager[860]: &lt;info&gt;  (ttyUSB2) closing serial port...
Jul 12 07:15:46 ubuntu modem-manager[860]: &lt;info&gt;  (ttyUSB2) serial port closed
Jul 12 07:15:46 ubuntu modem-manager[860]: &lt;info&gt;  (ttyUSB1) closing serial port...
Jul 12 07:15:46 ubuntu modem-manager[860]: &lt;info&gt;  (ttyUSB1) serial port closed
Jul 12 07:15:46 ubuntu NetworkManager[858]: &lt;warn&gt; (ttyUSB1): failed to look up interface index
Jul 12 07:15:46 ubuntu NetworkManager[858]: &lt;info&gt; WWAN now disabled by management service
Jul 12 07:15:46 ubuntu NetworkManager[858]: &lt;info&gt; (ttyUSB1): new GSM device (driver: 'qcserial' ifindex: -1)
Jul 12 07:15:46 ubuntu NetworkManager[858]: &lt;info&gt; (ttyUSB1): exported as /org/freedesktop/NetworkManager/Devices/3
Jul 12 07:15:46 ubuntu NetworkManager[858]: &lt;info&gt; (ttyUSB1): now managed
Jul 12 07:15:46 ubuntu NetworkManager[858]: &lt;info&gt; (ttyUSB1): device state change: 1 -&gt; 2 (reason 2)
Jul 12 07:15:46 ubuntu NetworkManager[858]: &lt;info&gt; (ttyUSB1): deactivating device (reason: 2).
Jul 12 07:15:46 ubuntu NetworkManager[858]: &lt;info&gt; (ttyUSB1): device state change: 2 -&gt; 3 (reason 0)
Jul 12 07:15:52 ubuntu NetworkManager[858]: &lt;info&gt; Activation (ttyUSB1) starting connection '3 Abbonamento'
Jul 12 07:15:52 ubuntu NetworkManager[858]: &lt;info&gt; (ttyUSB1): device state change: 3 -&gt; 4 (reason 0)
Jul 12 07:15:52 ubuntu NetworkManager[858]: &lt;info&gt; Activation (ttyUSB1) Stage 1 of 5 (Device Prepare) scheduled...
Jul 12 07:15:52 ubuntu NetworkManager[858]: &lt;info&gt; Activation (ttyUSB1) Stage 1 of 5 (Device Prepare) started...
Jul 12 07:15:52 ubuntu NetworkManager[858]: &lt;info&gt; (ttyUSB1): device state change: 4 -&gt; 6 (reason 0)
Jul 12 07:15:52 ubuntu NetworkManager[858]: &lt;info&gt; Activation (ttyUSB1) Stage 1 of 5 (Device Prepare) complete.
Jul 12 07:15:52 ubuntu NetworkManager[858]: &lt;info&gt; Activation (ttyUSB1) Stage 1 of 5 (Device Prepare) scheduled...
Jul 12 07:15:52 ubuntu NetworkManager[858]: &lt;info&gt; Activation (ttyUSB1) Stage 1 of 5 (Device Prepare) started...
Jul 12 07:15:52 ubuntu NetworkManager[858]: &lt;info&gt; (ttyUSB1): device state change: 6 -&gt; 4 (reason 0)
Jul 12 07:15:52 ubuntu NetworkManager[858]: &lt;info&gt; Activation (ttyUSB1) Stage 1 of 5 (Device Prepare) complete.
Jul 12 07:15:52 ubuntu modem-manager[860]: &lt;info&gt;  (ttyUSB1) opening serial port...
Jul 12 07:15:52 ubuntu modem-manager[860]: &lt;info&gt;  Modem /org/freedesktop/ModemManager/Modems/1: state changed (disabled -&gt; enabling)
Jul 12 07:15:53 ubuntu modem-manager[860]: &lt;info&gt;  Modem /org/freedesktop/ModemManager/Modems/1: state changed (enabling -&gt; enabled)
Jul 12 07:15:53 ubuntu NetworkManager[858]: &lt;info&gt; WWAN now enabled by management service
Jul 12 07:15:53 ubuntu modem-manager[860]: &lt;info&gt;  Modem /org/freedesktop/ModemManager/Modems/1: state changed (enabled -&gt; registered)
Jul 12 07:15:53 ubuntu modem-manager[860]: &lt;info&gt;  Modem /org/freedesktop/ModemManager/Modems/1: state changed (registered -&gt; connecting)
Jul 12 07:15:53 ubuntu modem-manager[860]: &lt;info&gt;  Modem /org/freedesktop/ModemManager/Modems/1: state changed (connecting -&gt; connected)
Jul 12 07:15:53 ubuntu NetworkManager[858]: &lt;info&gt; Activation (ttyUSB1) Stage 2 of 5 (Device Configure) scheduled...
Jul 12 07:15:53 ubuntu NetworkManager[858]: &lt;info&gt; Activation (ttyUSB1) Stage 2 of 5 (Device Configure) starting...
Jul 12 07:15:53 ubuntu NetworkManager[858]: &lt;info&gt; (ttyUSB1): device state change: 4 -&gt; 5 (reason 0)
Jul 12 07:15:53 ubuntu NetworkManager[858]: &lt;info&gt; Activation (ttyUSB1) Stage 2 of 5 (Device Configure) successful.
Jul 12 07:15:53 ubuntu NetworkManager[858]: &lt;info&gt; Activation (ttyUSB1) Stage 3 of 5 (IP Configure Start) scheduled.
Jul 12 07:15:53 ubuntu NetworkManager[858]: &lt;info&gt; Activation (ttyUSB1) Stage 2 of 5 (Device Configure) complete.
Jul 12 07:15:53 ubuntu NetworkManager[858]: &lt;info&gt; Activation (ttyUSB1) Stage 3 of 5 (IP Configure Start) started...
Jul 12 07:15:53 ubuntu NetworkManager[858]: &lt;info&gt; (ttyUSB1): device state change: 5 -&gt; 7 (reason 0)
Jul 12 07:15:53 ubuntu NetworkManager[858]: &lt;info&gt; starting PPP connection
Jul 12 07:15:53 ubuntu NetworkManager[858]: &lt;info&gt; pppd started with pid 2676
</code></pre></p>
<p>
Loader still works but you have to start it after a cold boot, also I noticed any attempt to start again the loader will crash the device, this also happens if you are using wvdial with the incorrect ttyUSBx port.
</p>
<p>
The best this is to start the loader after the computer booted, then use the network manager to connet skipping the use of wvdial.
</p>
<p>
I also noticed sometimes the devices hangs and reboot, so you have to start again the loader.
</p>
<p>
Using this notebook with ubuntu and linux, I got another issue. Sometimes when working with windows, after the computer is rebooted the wlan card is switched off and so it stays also when this notebook is booting linux making the device discovery useless. I think this problem is related with the capabilities to switch off the card via software and ubuntu 11 is not able to switch on it. If you are experiencing some problems with the loader check the wifi status led, to get the whole procedure working it must be blue while linux is booting.
</p>
