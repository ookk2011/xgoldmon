xgoldmon
===========

xgoldmon is a small tool to convert the messages output by the USB
logging mode of phones with Intel/Infineon XGold baseband processor
back to the GSM/UMTS radio messages sent over the air so you can watch
them in e.g. Wireshark in realtime.
This includes signalling for calls, SMS, USSD, paging for your and
other phones and so on.

Currently, these devices are supported:
- Samsung Galaxy S4     GT-I9500 (this is the version without LTE!)
- Samsung Galaxy S3     GT-I9300
- Samsung Galaxy Nexus  GT-I9250 (has to be rooted!)
- Samsung Galaxy S2     GT-I9100
- Samsung Galaxy Note 2 GT-N7100

Patches for other devices or to decode more messages are very much
welcome!

-Tobias Engel

Mail: tobias@sternraute.de
Twitter: @2b_as

Update for Modmobmap
---------------------

This update is a small adaptation to be run with [Modmobmap that was presented at BeeRump 2018](https://www.rump.beer/2018/slides/modmobmap.pdf). In this update was added small parser of log print taken from the XGold DIAG interface, that only displays cell logs information for 3G cells as follows: 

```bash
  ./xgoldmon -t s3 -m /dev/ttyACM1
```

The logs could then be read on the FIFO file created for that:
``` 
  $ cat ./celllog.fifo
  [...]
  [CellInfo]:PLMN=208-15;RAC=0x1;LAC=0x4e71;CID=0x1f****;DL_UARFCN=10737;UL_ARFCN=9787
  [CellInfo]:PLMN=208-20;RAC=0x1;LAC=0x4e71;CID=0x1f****;DL_UARFCN=2950;UL_ARFCN=2725
  [...]
  [CellInfo]:PLMN=208-20;RAC=0x1;LAC=0xb5aa;CID=0x97****;DL_UARFCN=10639;UL_ARFCN=9689
  [CellInfo]:PLMN=208-10;RAC=0x1;LAC=0xb5aa;CID=0x97****;DL_UARFCN=65535;UL_ARFCN=2850
  [...]
```
Note that secret code for ServiceMode *0011* should be typed before. 

To use it out-of-the-box (without Modmobmap), you could also connect to the AT interface as follows and change network operators as well as other things:

```bash
  screen /dev/ttyACM0 115200
  AT+COPS=1,2,"<operators>"
  [...]
```

How to build
---------------------

xgoldmon has been tested on Linux and OSX.

As a prerequisite, a recent version of libosmocore has to be
installed. See here for instructions:
http://bb.osmocom.org/trac/wiki/libosmocore (If you install it in a
non-standard location please set PKG_CONFIG_PATH accordingly.)

Then run "make" in the xgoldmon directory. An xgoldmon binary should
be created.

Before running xgoldmon
------------------------

To enable the logging mode ("diag mode") on the S2, S3 and Note2:
- Go to the Phone application, enter *#9900# and set "Debug Level
  Enabled" to "HIGH". The phone will reboot.
- Go to the Phone application again, enter *#7284# and set "USB" to
  "MODEM" and tap "SAVE and RESET". The phone will reboot again.

The Galaxy Nexus has to be rooted first to activate diag mode! Then:
- In the adb shell, as root, enter:
  echo MODEM > /sys/devices/tuna_otg/usb_sel
- Connect to the first of the serial devices (e.g. /dev/ttyACM0) with
  a terminal emulator and enter
  AT+TRACE=1

When connecting the phone via USB to the computer, several new
pseudo-tty devices should be created. The one with the second lowest
number should be the logging port. So for example on Linux, if you
have no other ttyACM* devices, it should be /dev/ttyACM1.

xgoldmon tries to set proper serial attributes on the device if the
"-s" option is specified. If that fails, you might have to do that
yourself with something like:


```bash
  stty 115200 pass8 raw -noflsh -F /dev/ttyACM1
```

Running xgoldmon
---------------------

E.g.:

```bash
  xgoldmon -t s3 -l /dev/ttyACM1
```

Full usage:
```bash
usage: ./xgoldmon [-t <phone type>] [-l] [-s] [-i <ip address>] [-v] <logfile or device>
  -t: select 's4', 's3', 'gnex', 's2' or 'note2' (default: 's3')
  -l: print baseband log messages
  -m: print Cell logs (could be used with Modmobmap)
  -s: set proper serial device attributes
  -i: send gsmtap packets to given ip address (default: 'localhost')
  -v: show debugging messages (more than once for more messages)
```

In some situations, the phone might close the device, causing xgoldmon
to exit. If you want to do some unsupervised logging, it might be a
good idea to put the call to xgoldmon in a loop.


Watching the radio messages in Wireshark
-----------------------------------------

xgoldmon uses libosmocore to send the radio messages in GSMTAP format
(http://bb.osmocom.org/trac/wiki/GSMTAP) to UDP port 4729 on the local
host. In order to monitor the packages with Wireshark, something has
to listen on that port, e.g.

```bash
  $ nc -u -l 4729
```

Then, in Wireshark, start a capture on the loopback interface. To see
only the GSMTAP messages, set this filter:

```
  udp.port==4729
```

GSM messages will be decoded out-of-the box in Wireshark. For UMTS/RRC
messages, you need a recent development version of Wireshark (at least
revision 47792), which you most likely will have to build yourself.

If everything works, it should look a bit like the
"screenshot-mtsms-while-in-a-call.png".
It contains a screenshot of Wireshark that shows an S3 receiving a
text message while in a call. (Lots of messages filtered out to show
the more relevant messages)


Thanks
-------

Many thanks to...

* Harald Welte and GSMK Cryptophone for their support in making this
  tool

* Nico Golde (@iamnion) for adding support for the Note 2

* Yan Grunenberger (@yangrunenberger) for finding out how to activate
  diag mode on the Galaxy Nexus

* Max for adding options to set serial attributes and gsmtap ip address
