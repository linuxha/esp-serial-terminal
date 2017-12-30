# esp-serial-terminal
==========================
This is a fork and translation of Dima-Ch's [ESP-Serial-Terminal](https://github.com/dima-ch/esp-serial-terminal).

ESP8266 WiFi - RS232 bridge. An additional function is relay control. Applicable for shorting the contacts of the PC power button.
Designed to implement a remote terminal for a home server. Allows you to work with the system before loading network interfaces and the possibility of using SSH / VNC.

For example, managing grub, entering the password from the encrypted root partition, solving the problems of mounting FS failure, or simply ttyS console.

The firmware is based on the firmware found at [ESP8266-transparent-bridge project](https://github.com/beckdac/ESP8266-transparent-bridge).
We used the ESP8266 KiCAD library [techinc-kicad-lib](https://github.com/techinc/techinc-kicad-lib).

The assembled firmware in [releases](https://github.com/dima-ch/esp-serial-terminal/releases).

## Hardware
Currently, the hardware is available for the ESP8266 ESP-03. The circuit and the printed circuit board in KiCAD format are in the schematic directory. The GPIO2 module is connected to the indication LED.
GPIO14 is connected to the relay. The GPIO14 is selected because the logic level on it does not jump when the ESP8266 is turned on.

## Indication
* LED blinks during operation
* When the client is connected, the LED blinks more slowly

Connectivity is conveniently done by telnet clients.
ConnectBot for Android works with the default settings, the Linux telnet utility requires the environment variable export TERM = VT100, the -8 option and execution after connecting the ^] mode character.

## Configuration
Make sure that the code is set to #define CONFIG_DYNAMIC

Configuration and control AT commands

```
+++ AT # output in response to OK
+++ AT PWBTN <duration: SHORT, LONG, HARDRESET>
# Relay contacts management: SHORT - closure by 0.5s, LONG - by 4s, HARDRESET - 4c, pause in 1c, then 0.5s
+++ AT MODE # output the current mode
+++ AT MODE <mode: 1 = STA, 2 = AP, 3 = both> # set the mode
+++ AT STA # display the current ssid and connection password for the access point
+++ AT STA <ssid> <password> # set ssid and password for access point
+++ AP AT # display the current access point settings
+++ AT AP <ssid> # set the mode of the open access point with the specified ssid
+++ AT AP <ssid> <pw> [<authmode> [hide-ssid [<ch>]]]]
# set the access point parameters, authmode: 1 = WEP, 2 = WPA, 3 = WPA2,4 = WPA + WPA2,										 # hide-ssid:1-hide, 0-show(not hide), channel: 1..13
+++ AT BAUD # display the current UART settings
+++ AT BAUD <baud> [data [parity [stop]]] # set bitrate and optionally data bits = 5/6/7/8, parity = N / E / O, stop bits = 1 / 1.5 / 2
+++ AT PORT # display the current TCP port
+++ AT PORT <port> # set the TCP port (reboot)
+++ AT FLASH # output save settings
+++ AT FLASH <1 | 0> # 1: changing the UART settings (++ AT BAUD ...) is saved (by default), 0: changes are not saved
+++ AT RESET # software reset
```
If successful, all commands return "OK" or the corresponding output. Note that the password and ssid can not contain spaces.

An example of configuration. telnet is used without keys and settings, unlike the mode of operation. If there is no response to the +++ AT command with manual input, I recommend inserting a command into the terminal from the clipboard.

```
user@host:~$ telnet 192.168.1.197
Trying 192.168.1.197...
Connected to 192.168.1.197.
Escape character is '^]'.
+++AT MODE
MODE=3
OK
+++AT AP
SSID=ESP_9E2EA6 PASSWORD= AUTHMODE=0 IS_HIDDEH_SSID=0 CHANNEL=3
OK
+++AT AP newSSID password
OK
+++AT AP
SSID=newSSID PASSWORD=password AUTHMODE=2 IS_HIDDEH_SSID=0 CHANNEL=3
OK
+++AT AP ESP_9E2EA6
OK
+++AT AP
SSID=ESP_9E2EA6 PASSWORD= AUTHMODE=0 CHANNEL=3
OK
^]c

telnet> c
Connection closed.
```
Firmware:
```
esptool.py --port /dev/ttyXXX write_flash 0x00000 0x00000.bin 0x40000 0x40000.bin
```
or using ESP8266Flasher.exe from https://github.com/nodemcu/nodemcuflasher with
0x00000.bin at 0x00000
0x40000.bin at 0x40000

For more information, see the [ESP8266-transparent-bridge project](https://github.com/beckdac/ESP8266-transparent-bridge)

## TODO

* an error on the PCB and the circuit, the DB9 pins are reversed
* Reduce clients to 2 and increase the send buffer
