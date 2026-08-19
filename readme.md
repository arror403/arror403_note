### USB DAC issue


#### Part 1 Too loud

As an audiophile, I use Audioquest's Dragonfly Cobalt. After I changed to openSUSE from Windows, its output is too loud even when set in the minimum volume. 

Quick fix: Use alsamixer to configure in driver level. But it would need manual set every time.

How to automate it? Use `udev`

```
sudo nano /etc/udev/rules.d/99-usb-dac.rules
```

Write this line:

```
ACTION=="add", SUBSYSTEM=="sound", ATTRS{idVendor}=="21b4", ATTRS{idProduct}=="0085", RUN+="/usr/bin/amixer -c v1 set PCM 4"
```

`ATTRS{idVendor}=="21b4", ATTRS{idProduct}=="0085" specify the Dragonfly Cobalt`

Use `lsusb` to find `XXXX:XXXX` part.

e.g. this is Apple's type-c to 3.5mm adapter:
Bus 003 Device 010: ID 05ac:110a Apple, Inc. USB-C to 3.5mm Headphone Jack Adapter


`/usr/bin/amixer -c v1 set PCM 4` : command that do the same thing as above quick fix

Use `man amixer` to look up its documentation.

#### Part 2 Sampling rate


Cobalt has a feature that its light will change according to the sampling rate of the file you're playing. 44.1 kHz is green, 48kHz is blue, 88.2kHz is orange, 96kHz is white

How to let Cobalt process it without resampling to 48kHz(default) all the time?

```
sudo nano /home/{your_user_name}/.config/pipewire/pipewire.conf.d/custom-rates.conf
```

Write this:

```
context.properties = {
    default.clock.allowed-rates = [ 44100 48000 88200 96000 ]
}
```

But this will arise another problem, all devices will follow this rule, how to set Cobalt the only that follow this rule?


#### Part 3 Specify device

```
sudo nano /home/{your_user_name}/.config/wireplumber/wireplumber.conf.d/99-dragonfly.conf
```


Write the following:

```
monitor.alsa.rules = [
    {
        # RULE 1: Lock ALL ALSA devices to 48kHz
        matches = [
        {
            node.name = "~alsa_output.*"
        }
        ]
        actions = {
        update-props = {
            audio.allowed-rates = [ 48000 ]
        }
        }
    }
    {
        # RULE 2: Override Rule 1 specifically for the DragonFly Cobalt
        matches = [
        {
            node.name = "~alsa_output.usb-AudioQuest_AudioQuest_DragonFly_Cobalt.*"
        }
        ]
        actions = {
        update-props = {
            audio.allowed-rates = [ 44100 48000 88200 96000 ]
        }
        }
    }
]
```

### Overview of ALSA, WirePlumber and PipeWire

```

  +-------------------------------------------+
  |               Applications                |  (Audacious, Spotify, Firefox)
  +-------------------------------------------+
                        |
  +-------------------------------------------+
  |        WirePlumber (The "Brain")          |  (Policies, Rules, Decisions)
  +-------------------------------------------+
                        |
  +-------------------------------------------+
  |         PipeWire (The "Engine")           |  (Mixing, Routing, Processing)
  +-------------------------------------------+
                        |
  +-------------------------------------------+
  |         ALSA (The "Foundation")           |  (Drivers, Hardware Access)
  +-------------------------------------------+
                        |
  +-------------------------------------------+
  |         DragonFly Cobalt (Hardware)       |
  +-------------------------------------------+
  
```


# TOEIC


![Average](https://github.com/arror403/arror403_note/blob/main/pics/aver.png)

### Average = 566

![Disributaion](https://github.com/arror403/arror403_note/blob/main/pics/dist.png)

### Percentage of people whose score >= 900 is 4.3%

![900+](https://github.com/arror403/arror403_note/blob/main/pics/norm_dist_900.png)

### With the above two, we can calculate standard deviation in normal distribution.

![815+](https://github.com/arror403/arror403_note/blob/main/pics/norm_dist_815.png)

### Put my score in.


> **Is 90% good? It depends.**

