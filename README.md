# simul-1k2-9k6
How to run both 1200 and 9600 baud packet simultaneously using one sound card on a single computer.

#### Goal
* To be able to transistion from infrastructure using 1200 baud
packet to 9600 baud packet.
  * Transparently handle 1200 or 9600 baud packet connections using a
  single computer (Raspberry Pi)
* If you want to manually switch between 1200 & 9600 baud packet this
functionality already works using [this script](
https://github.com/nwdigitalradio/n7nix/blob/master/baudrate/speed_switch.sh).
  * The script will change the baud rate of both direwolf channels to
  either 1200 or 9600.

#### Assumptions
* Using a Kenwood TM-V71a which runs 1200/9600 packet on the radio's 9600 baud packet setting.
  * No config changes on the radio are required when running either baud rate.
* Use direwolf sound modem with a sound card.
* One connection using one baud rate at a time

#### Currently working

* Both sound card channels are switched between
1200 & 9600 baud using [speed_switch.sh](https://github.com/nwdigitalradio/n7nix/blob/master/baudrate/speed_switch.sh)
script
  * Depends on the ```/etc/ax25/port.conf``` file configuration
  * This file is used to configure 1200 or 9600 baud rate for each of the sound card channels.

#### Methods to test
##### Listen on both direwolf ports

* Hack a single mDin6 cable from radio to each sound card audio port
  * This will monitor both 1200 baud & 9600 baud modems in two direwolf channels.
    * Run each of the two modem ports in Direwolf as either 1200 or
    9600 baud.
    * Should be able to tell which udrc/direwolf channel is active from direwolf
    decode.
  * Since both GPIO and transmit output pins are tied together, will
need to control transmit pins with alsa settings.
    * Assuming changes to alsa control lines not require a restart of alsa or a reboot.
  * Assign two SSIDs to use for either 1200 or 9600 baud in _ax25d.conf_

##### Control which baud rate to use with beacon

* Using a beacon on any valid band plan frequency send a command to set baud rate to use.
  * Need a mechanism to set default transmit channel to 1200 baud.
  * When 9600 baud transaction completes will monitor both channels
  but only transmit on 1200 baud channel.



##### Notes from John

You asked if we could run 1200/9600 baud through the same radio in parallel
(interleaved) on the same frequency.  This would benefit transitioning from
1200 baud to 9600 baud on a channel for services like Winlink.

Using pulseaudio [split-channels config](https://github.com/nwdigitalradio/split-channels) and
setting up sub channels for left and right channels in /etc/asound.conf

```
pcm.draws-capture-left {
  type pulse
  device "draws-capture-left"
}
pcm.draws-playback-left {
  type pulse
  device "draws-playback-left"
}
pcm.draws-capture-right {
  type pulse
  device "draws-capture-right"
}
pcm.draws-playback-right {
  type pulse
  device "draws-playback-right"
}


pcm.draws-capture-left-sub {
  type pulse
  device "draws-capture-left"
}
pcm.draws-playback-left-sub {
  type pulse
  device "draws-playback-left"
}
pcm.draws-capture-right-sub {
  type pulse
  device "draws-capture-right"
}
pcm.draws-playback-right-sub {
  type pulse
  device "draws-playback-right"
}
```

### direwolf.conf file
* I am able to configure direwolf.conf to access the sound channels --
```
ADEVICE0 draws-capture-right draws-playback-right

ACHANNELS 1
ARATE 48000

CHANNEL 0
MYCALL K7VE-1
MODEM 1200
PTT RIG 120 /dev/ttyUSB0

ADEVICE1 draws-capture-right-sub draws-playback-right-sub

ACHANNELS 1
ARATE 48000

CHANNEL 0
MYCALL K7VE-2
MODEM 9600
PTT GPIO 23
```

The compiled version (1.6 c) doesn't have hamlib included so the PTT RIG
doesn't work.  However, I have been able to transmit both 1200 and 9600
baud. (Due to being in a faraday cage, my hand held doesn't receive GPS and
is not beaconing, so there insufficient receive testing. -- however, we
have been able to run 9600 and 1200 through the FT-817 before).

For this application it would be helpful if PTT GPIO could be shared
between two 'channels', but with further work, with this particular
configuration, I could probably use a combination of RIGCTL and GPIO.  I
think an earlier version of Direwolf allowed me to run this test on a UDRC
sharing the GPIO.

In summary, we probably to could get there, but it would require either
modifying direwolf to allow a shared GPIO for PTT, or some RIG control by a
secondary mechanism.

##### Problems

* If connecting both DRAWS mini Din6 connectors to a single radio then
need to turn off one channel AFIN pin on sound card (PKD audio signal
for packet transmission on radio) when transmitting.
