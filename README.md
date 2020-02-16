# simul-1k2-9k6
How to run both 1200 and 9600 baud packet simultaneously using one sound card on a single computer.

#### Goal
* To be able to transistion from infrastructure using 1200 baud
packet to 9600 baud packet.
  * Transparently handle 1200 or 9600 baud packet connections using a
  single computer (Raspberry Pi)
* If you want to manually switch between 1200 & 9600 baud packet this
functionality already works by using [this script](
https://github.com/nwdigitalradio/n7nix/blob/master/debug/speed_switch.sh).
  * The script will change the baud rate of both direwolf channels to
  either 1200 or 9600.

#### Assumptions
* Using a Kenwood TM-V71a which runs 1200/9600 packet on the radio's 9600 baud packet setting.
  * No config changes on the radio are required when running either baud rate.
* Use direwolf sound modem with a sound card.
* One connection using one baud rate at a time

#### Currently working

* Both sound card channels are switched between
1200 & 9600 baud using [speed_switch.sh](https://github.com/nwdigitalradio/n7nix/blob/master/debug/speed_switch.sh)
script
  * Depends on the ```/etc/ax25/packet_9600baud```
file existing or not
* This method needs to be extended to include configuring 1200 or 9600 baud rate for each of the sound card channels.

#### Methods to test
##### Listen on both direwolf ports

* Hack a single mDin6 cable from radio to each sound card audio port
  * This will tie the AFOUT & DISCOUT lines together to monitor both 1200
  baud & 9600 baud modems in two direwolf channels.
    * Run each of the two modem ports in Direwolf as either 1200 or 9600
    * Should be able to tell which udrc channel to use from direwolf
    decode.
  * Since both GPIO and transmit output pins are tied together, will
need to control transmit pins with alsa settings.
    * Assuming I can make changes to alsa control lines & not require
    a restart of alsa or a reboot.
  * Assign two SSIDs to use for either 1200 or 9600 baud in _ax25d.conf_

##### Control which baud rate to use with beacon

* Using a beacon on any valid band plan frequency send a command to set baud rate to use.
  * Need a mechanism to set default transmit channel to 1200 baud.
  * When 9600 baud transaction completes will monitor both channels
  but only transmit on 1200 baud channel.


##### Problems

* If connecting both DRAWS mini Din6 connectors to a single radio then
need to turn off one channel AFIN pin on sound card (PKD audio signal
for packet transmission on radio) when transmitting.
