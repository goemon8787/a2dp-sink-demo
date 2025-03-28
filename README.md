# a2dp-sink-demo
動いて

# 動かない
bluetoothをコンテナ側から乗っ取るのはかなりきつそう

# ネイティブで構築
## Environment
- OS: Linux raspberrypi 6.12.20-v8-16k+ #1867 SMP PREEMPT Wed Mar 26 12:49:06 GMT 2025 aarch64 GNU/Linux

## Installation
```
sudo apt install rfkill
# bluez, pulseaudio, alsa, pipewireなどプリインストール済み
```

## Configuration
/lib/systemd/system/bluetooth.service
```diff
+ ExecStart=/usr/libexec/bluetooth/bluetoothd --plugin=a2dp --compat --noplugin=sap
- ExecStart=/usr/libexec/bluetooth/bluetoothd
```

/etc/bluetooth/main.conf
```diff
+ Disable=headset
+ Class 0x20041c
```

/etc/pulse/daemon.conf
```diff
+ resample-method = trivial
+ default-sample-format = s32le
+ default-sample-rate = 384000
+ default-fragments = 8
+ ; default-fragment-size-msec = 15
+ default-fragment-size-msec = 125
---
- default-fragment-size-msec = 15
```

/etc/pulse/default.pa
```diff
+ #load-module module-switch-on-port-available
- load-module module-switch-on-port-available
+ set-default-sink alsa_output.usb-GuangZhou_FiiO_Electronics_Co._Ltd_FiiO_K7-00.iec958-stereo
```

/usr/share/pipewire/pipewire.conf (bk忘れた)
```diff
+ default.clock.rate = 384000
+ #default.clock.allowed-rates = [ 384000 ]
+ default.clock.min-quantum = 32
```

## Setup
### Services Status
```shell
sudo systemctl status dbus
sudo systemctl status bluetooth
systemctl --user status pulseaudio
systemctl --user status pipewire
systemctl --user status pipewire-pulse
```
→ Check All status are loaded

### Recognize Devices
```shell
lsusb
```
```

Bus 001 Device 003: ID 2357:0604 TP-Link TP-Link Bluetooth USB Adapter
Bus 001 Device 002: ID 2972:0047 FiiO Electronics Technology FiiO K7
...
```
→ Displaying audio and extra bt devices.

### Bluetooth Devices
```
hciconfig
```
```
hci1:   Type: Primary  Bus: USB
        BD Address: 8C:90:2D:42:09:C5  ACL MTU: 1021:6  SCO MTU: 255:12
        UP RUNNING PSCAN
        RX bytes:7089 acl:139 sco:0 events:392 errors:0
        TX bytes:35616 acl:134 sco:0 commands:222 errors:0

hci0:   Type: Primary  Bus: UART
        BD Address: 2C:CF:67:93:8C:6A  ACL MTU: 1021:8  SCO MTU: 64:1
        UP RUNNING PSCAN
        RX bytes:6577745 acl:16272 sco:0 events:557 errors:0
        TX bytes:68955 acl:129 sco:0 commands:437 errors:0
```
→ if status is `DOWN`, execute `sudo hciconfig ${hciN} up` and status is able to be `UP [RUNNING] PSCAN [...]`

### Selected Audio Sink
```shell
pactl list sinks | grep -B1 -A9 State:
```
```
Sink #66
        State: SUSPENDED
        Name: alsa_output.usb-GuangZhou_FiiO_Electronics_Co._Ltd_FiiO_K7-00.iec958-stereo
        Description: FiiO K7 Digital Stereo (IEC958)
        Driver: PipeWire
        Sample Specification: s32le 2ch 384000Hz
        Channel Map: front-left,front-right
        Owner Module: 4294967295
        Mute: no
        Volume: front-left: 16384 /  25% / -36.12 dB,   front-right: 16384 /  25% / -36.12 dB
                balance 0.00
```

### Bluetooth Pairing
```shell
bluetoothctl

# bluetoothctl interactive console
[bluetooth]# list
--
Controller 2C:CF:67:93:8C:6A [default]
Controller 8C:90:2D:42:09:C5
--

[bluetooth]# select 2C:CF:67:93:8C:6A
[bluetooth]# show
---
Controller 2C:CF:67:93:8C:6A (public)
        Name: BlueZ 5.66
        Alias: BlueZ 5.66
        Class: 0x006c0000
        Powered: no
        Discoverable: no
        DiscoverableTimeout: 0x000000b4
        Pairable: yes
        UUID: Audio Source              (0000110a-0000-1000-8000-00805f9b34fb)
        UUID: PnP Information           (00001200-0000-1000-8000-00805f9b34fb)
        UUID: Audio Sink                (0000110b-0000-1000-8000-00805f9b34fb)  # IMPORTANT
        UUID: Handsfree Audio Gateway   (0000111f-0000-1000-8000-00805f9b34fb)
        UUID: Generic Access Profile    (00001800-0000-1000-8000-00805f9b34fb)
        UUID: Generic Attribute Profile (00001801-0000-1000-8000-00805f9b34fb)
        UUID: Device Information        (0000180a-0000-1000-8000-00805f9b34fb)
        UUID: Handsfree                 (0000111e-0000-1000-8000-00805f9b34fb)
        Modalias: usb:v1D6Bp0246d0542
        Discovering: no
        Roles: central
        Roles: peripheral
Advertising Features:
        ActiveInstances: 0x00 (0)
        SupportedInstances: 0x05 (5)
        SupportedIncludes: tx-power
        SupportedIncludes: appearance
        SupportedIncludes: local-name
---

[bluetooth]# system-alias pi1
---
Changing pi1 succeeded
[CHG] Controller 2C:CF:67:93:8C:6A Alias: pi1
---

[bluetooth]# agent DisplayYesNo
---
Agent is already registered
---

[bluetooth]# default-agent
---
Default agent request successful
---

[bluetooth]# power on
---
[CHG] Controller 2C:CF:67:93:8C:6A Class: 0x006c0000
Changing power on succeeded
[CHG] Controller 2C:CF:67:93:8C:6A Powered: yes
---

[bluetooth]# discoverable on
---
Changing discoverable on succeeded
[CHG] Controller 2C:CF:67:93:8C:6A Discoverable: yes
---

###
# pairing from audio source devices
###

[NEW] Device F8:10:93:83:9F:B0 Goemon
Request confirmation

###
# type yes multiply
###

---
[agent] Confirm passkey 551814 (yes/no): yes
.
.
.
[agent] Authorize service 0000111e-0000-1000-8000-00805f9b34fb (yes/no): yes
---

[Goemon]# devices Paired
---
Device F8:10:93:83:9F:B0 Goemon
---

[Goemon]# trust F8:10:93:83:9F:B0
---
[CHG] Device F8:10:93:83:9F:B0 Trusted: yes
Changing F8:10:93:83:9F:B0 trust succeeded
---
```
→ setting all audio source and sink

### sound check
```
# play from raspberry pi
aplay xxx.wav
```
→ and play audio from any audio source

## Troubleshooting
### USE BT Receiver is not UP, or can't change power on.
- UP from hciconfig
```
sudo systemctl ${hciN} reset
sudo systemctl ${hciN} up
```

- reset driver
```
sudo hciconfig ${hciN} down
sudo rmmod btusb
sudo modprobe btusb
sudo hciconfig ${hciN} up
```

- check system log
```
dmesg | grep hci
```