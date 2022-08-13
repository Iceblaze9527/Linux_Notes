---
tags: Linux
---

# Raspberry Pi Assortment
## 1. Enable SSH Connection and Auto Wireless Connection upon Boot
### 1.1 Summary
```bash
$ cd /Volumes/boot
$ touch ssh
$ sudo nano wpa_supplicant.conf
$ ssh pi@raspberrypi.local # or ip address
$ sudo nano /etc/hosts
$ sudo nano /etc/hostname
$ sudo shutdown -h now
```

### 1.2 After flashing the image
Reinsert the SD card into your Mac and you'll see a new externel device named `boot`. Create an empty file named `ssh` under `boot/` and a config file named `wpa_supplicant.conf` with the following content: 
```json
country=CN
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
	ssid="Tsinghua-Secure"
	psk="password"
	key_mgmt=WPA-PSK
	priority=1
	} 
```

You can setup multiple networks available for RPi by adding another `network={}` in the configuration file. The ones with larger priority numbers come first.

Safely eject the SD device and insert it into RPi. Power on and the system will boot automatically. A green LED shall be on if everything goes smoothly.

## 2. SSH into RPi
Usually, the default username for a new RPi is `pi`, with hostname `raspberrypi` and password `raspberry`. So, ==under the same network specified in the config file==, first try on your SSH client `ssh pi@raspberrypi.local`, then SSH login should be prompted if your device can find your RPi's IP address successfully; otherwise, you will need to find its IP first.

### 2.1 [Find the IP Address for RPi (if necessary)](https://pimylifeup.com/raspberry-pi-ip-address/)
Several possible ways to find your RPi's IP address (ranked by priority):

- If available, log in to your network router to try finding its client list, you will see a client with hostname `raspberrypi`
 and its IP
- Try `ping raspberrypi.local` to retrieve its IP address
- Use `nmap` (this may not work though)

Then you can SSH into your RPi by `ssh pi@ip_address`.

### 2.2 Change Hostname (Permanently) and Password
The default password should be changed asap for security concerns once you login successfully. Use `passwd` to set a new one.

If you have multiple RPi hosts working simutaneously, then different hostnames should be set to distinguish them. Use `sudo nano /etc/hosts`, and typically you will see a line: `127.0.1.1    raspberrypi`. Change `raspberrypi` to whatever you want. The same goes for `/etc/hostname`. Reboot the system using `sudo reboot` to make it work.

Use `ssh pi@new_hostname.local` and the new password the next time you log in to the RPi system.

### 2.3 Safely Shutdown RPi
As you may have already noticed, there is no button to switch the Raspberry Pi on or off. Your first instinct is probably to pull the power cord, but this is highly not recommended.

There are many reasons why pulling the power cord while your operating system is still running is not a good idea.

- Firstly, by pulling the power cord out early, you heighten the risk of your SD card becoming corrupt. ==(This is real)==
- Secondly, anything that is running will not make a graceful exit and save. This forced exit may cause data loss depending on what your Raspberry Pi was doing at the time.

The easiest way to shutdown the Raspberry Pi correctly is to use a very simple command. You can find the command right below.

`sudo shutdown -h now`

It will do the following process to ensure the operating system shutdowns gracefully.

1. It will send `SIGTERM` to all the running processes, so they can save and exit gracefully.
2. After an interval, it sends `SIGKILL`, so that any remaining processes will be halted.
3. Lastly, it will unmount all the file systems.
4. The screen will now show System Halted.
5. You can now remove the power cord with minimal risk to your Raspberry Pi and the operating system.
6. To start the Raspberry Pi simply plug back in the USB power cord.

> Note: You can use a shorter alias (e.g. `shut`) to avoid typing the command every time. `sudo nano ~/.bashrc` and add a line `alias alias_command='original_command'`, then `source ~/.bashrc`.

## 3. [Wi-Fi Config using `wpa_supplicant`](https://cloud.tencent.com/developer/article/1379709?from=information.detail.linux%20%E8%87%AA%E5%8A%A8%E8%BF%9E%E6%8E%A5wifi)

### 3.0 Current Connection
```bash
$ iw dev
phy#0
        Unnamed/non-netdev interface
                wdev 0x2
                addr 22:67:d4:2d:ca:d3
                type P2P-device
                txpower 31.00 dBm
        Interface wlan0# Wi-Fi card name
                ifindex 2
                wdev 0x1
                addr b8:27:eb:55:5d:e1
                ssid Tenda_4334F0# Current Connection
                type managed
                channel 8 (2447 MHz), width: 20 MHz, center1: 2447 MHz
                txpower 31.00 dBm
```

### 3.1 Scan Networks
```bash
$ wpa_cli -i wlan0 scan
OK
$ wpa_cli -i wlan0 scan_result
c8:3a:35:43:34:f0       2447    -31     [WPA-PSK-CCMP][ESS]     Tenda_4334F0
18:69:da:81:5a:f8       2422    -59     [WPA-PSK-CCMP+TKIP][WPA2-PSK-CCMP+TKIP][ESS]    CMCC-E2DT
ec:26:ca:7a:58:04       2437    -61     [WPA-PSK-CCMP][WPA2-PSK-CCMP][ESS]      TP-LINK_2429
6c:16:32:c2:9c:08       2462    -64     [WPA-PSK-CCMP+TKIP][WPA2-PSK-CCMP+TKIP][WPS][ESS]       lnunicom_7Zjn
00:66:19:f3:5f:71       2412    -70     [WPA2-PSK-CCMP][WPS][ESS]
00:66:19:f3:5f:70       2412    -71     [WPA-PSK-CCMP][WPA2-PSK-CCMP][WPS][ESS] ChinaUnicom-33333
00:66:19:f3:5f:75       2412    -71     [WPA2-PSK-CCMP][WPS][ESS]
c8:c2:fa:17:47:7c       2437    -86     [WPA2-PSK-CCMP][WPS][ESS]       HUAWEI-FFZ389
ba:80:35:4c:f0:db       2462    -87     [WPA2-PSK-CCMP][ESS]
ba:80:35:4c:72:7e       2462    -90     [WPA2-PSK-CCMP][ESS]
7c:03:c9:32:9c:9a       2452    -91     [WPA-PSK-CCMP+TKIP][WPA2-PSK-CCMP+TKIP][WPS][ESS]       ChinaNet-3F4z
```

### 3.2 Add Networks
```bash
$ wpa_cli -i wlan0 add_network #添加一个网络连接,并返回网络ID号
0
$ wpa_cli -i wlan0 set_network 0 ssid '"SSID"'
OK
$ wpa_cli -i wlan0 set_network 0 psk '"password"' # $ wpa_cli -i wlan0 set_network 0 key_mgmt NONE; if no password is needed
OK
$ wpa_cli -i wlan0 set_network 0 priority 2
OK
$ wpa_cli -i wlan0 set_network 0 scan_ssid 1
OK
``` 

### 3.3 Connect Networks
```bash
$ wpa_cli -i wlan0 enable_network 0
$ wpa_cli -i wlan0 select_network 0 # cancel previous connection
$ sudo dhcpcd wlan0
```

### 3.4 Save Networks
```bash
$ wpa_cli -i wlan0 save_config #save current config to wpa_supplicant.conf
$ sudo nano /etc/wpa_supplicant/wpa_supplicant.conf # directly modify the file
```

### 3.3 Monitor Networks
```bash
$ wpa_cli -i wlan0 status
bssid=c8:3a:35:43:34:f0
freq=2447
ssid=Tenda_4334F0
id=1
mode=station
pairwise_cipher=CCMP
group_cipher=CCMP
key_mgmt=WPA-PSK
wpa_state=COMPLETED
ip_address=192.168.0.100
p2p_device_address=52:e6:89:83:d5:4d
address=b8:27:eb:55:5d:e1
uuid=d0b9ba58-70bc-5d7d-b62f-1becbe057303
```
```bash
$ wpa_cli -i wlan0 list_network
network id / ssid / bssid / flags
0       ITSHUAWEIP9     any
1       Tenda_4334F0    any     [CURRENT]
```

### 3.4 Disable and Remove Networks
```bash
$ wpa_cli -i wlan0 disable_network 0
OK
$ wpa_cli -i wlan0 remove_network 0 #disable first
OK
$ wpa_cli -i wlan0 save_config
OK
```

## 4. Audio Config
### 4.0 Introduction to ALSA on RPi
ALSA (Advanced Linux Sound Architecture) is a low-level framework that carries the audio system on Raspberry Pi which provides kernel drivers for its external playback or recording devices. The framework also contains programs for audio applications and some hands-on command line utilities.

### 4.1 Find Audio Device
```bash
$ lsusb #list usb device (for usb sound card)
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 003: ID 0c76:161f JMTek, LLC. # we will use this one
Bus 001 Device 002: ID 2109:3431 VIA Labs, Inc. Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

```bash
$ cat /proc/asound/cards #check sound cards
 0 [Headphones     ]: bcm2835_headphonbcm2835 Headphones - bcm2835 Headphones
                      bcm2835 Headphones
 1 [Device         ]: USB-Audio - USB PnP Audio Device # we will use this one
                      USB PnP Audio Device at usb-0000:01:00.0-1.3, full speed
```

```bash
$ sudo aplay -l #check playable device
**** List of PLAYBACK Hardware Devices ****
card 0: Headphones [bcm2835 Headphones], device 0: bcm2835 Headphones [bcm2835 Headphones]
  Subdevices: 8/8
  Subdevice #0: subdevice #0
  Subdevice #1: subdevice #1
  Subdevice #2: subdevice #2
  Subdevice #3: subdevice #3
  Subdevice #4: subdevice #4
  Subdevice #5: subdevice #5
  Subdevice #6: subdevice #6
  Subdevice #7: subdevice #7
card 1: Device [USB PnP Audio Device], device 0: USB Audio [USB Audio]# memorize this card and device number
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

### 4.2 Switch Default Audio Device
Modify `/etc/asound.conf` (system level) or `~/.asoundrc` (user level)

```bash
$ sudo nano ~/.asoundrc # for example
# add
defaults.ctl.card 1 
defaults.pcm.card 1
defaults.pcm.device 0
```

### 4.3 Test Audio Device
```bash
$ speaker-test -c2 -t wav
```

### 4.4 Play `.mp3` Files using `sox`
```bash
$ sudo apt-get install sox
$ sudo apt-get install libsox-fmt-mp3
$ sox test.mp3 -d
```

[More for `sox` here](https://www.jianshu.com/p/be8977de4a6b)

### 4.5 Adjust Volume (Interactive GUI)
```bash
alsamixer
```