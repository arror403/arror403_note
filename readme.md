*Power by nature intelligence*


[BIOS update](https://github.com/arror403/arror403_note#bios-update)

[DNS](https://github.com/arror403/arror403_note#dns)

[USB DAC issue](https://github.com/arror403/arror403_note#usb-dac-issue)

[TOEIC analysis](https://github.com/arror403/arror403_note#usb-dac-issue)

---

### BIOS update

What's new in version 1.17? According to [Acer](https://community.acer.com/en/kb/articles/18840-update-your-secure-boot-certificates-in-june-2026-to-stay-protected):

> Microsoft Secure Boot certificates that were originally issued in 2011 are approaching expiration starting in June 2026.

Acer didn't provide it on its [website](https://www.acer.com/tw-zh/support/product-support/PH315-54/downloads). Only a pdf file:

![](https://github.com/arror403/arror403_note/blob/main/pics/Screenshot_20261011_113103.png)

I'm dual booting Windows for such scenario. But to my surprise, I didn't get the update by Windows Update. The reason? I'm using a specialized version of Windows that only get crucial updates and update for firmware is not provided. (BTW, I did so because Windows Update is notorious for breaking Windows and pushing unnecessary updates.)

Then, how should I update it? I tried using `fwupdmgr` on my openSUSE part. Not provided, either. Understandable, since it's a certificate of MS secure boot. Linux is totally fine working without it.

One thing comes up in my mind. Microsoft has a [website](https://catalog.update.microsoft.com/Home.aspx) that include dedicate files that are used for Windows Update.

I was thinking by simply searching `5.43.1.17` (version number, the old one is `5.43.1.15`) and I can find files that usually would be provided by vendor. BUT, the same version number was used for different products.

By some discussion with chatGPT it helped me to find the exact one I need:

Run it in PowerShell(admin)
```shell
Get-PnpDevice -PresentOnly | Where-Object { $_.Class -eq 'Firmware' } | Format-List Status,Class,FriendlyName,InstanceId
```
The output would be something like this:

```
Status       : OK
Class        : Firmware
FriendlyName : Acer Inc. System Firmware 1.15
InstanceId   : UEFI\RES_{93F76199-AB1E-4993-88B7-A5DB5309D6AA}\0
Status       : OK
Class        : Firmware
InstanceId   : UEFI\RES_{9AB49E83-DF0B-3A4D-93FA-7232A979A2DE}\0
Status       : OK
Class        : Firmware
InstanceId   : UEFI\RES_{9AB49E83-DF0B-3A4D-93FA-7232A979A2DF}\0
```

**`InstanceId: 93F76199-AB1E-4993-88B7-A5DB5309D6AA`** is the critical part.

By searching it:

![](https://github.com/arror403/arror403_note/blob/main/pics/Screenshot_20261011_115924.png)

`InstanceId` should match `Supported Hardware IDs`

Download it. It's a .cab file, can be extracted like .zip files. There are 3 in version `5.43.1.17` (you might got more). You can choose either of them. They're logically identical. I used [UEFITool](https://github.com/LongSoft/UEFITool) and [uefi-firmware-parser](https://github.com/theopolis/uefi-firmware-parser) to find out the difference is `signing`. You might noticed older versions have more than one release, too.

![](https://github.com/arror403/arror403_note/blob/main/pics/Screenshot_20261011_120653.png)

Open PowerShell in admin mode and do:

```shell
pnputil /add-driver X:\{path_to_the_folder}\BIOS_1.17.INF /install
```
Restart your laptop/PC. That's the part you'll got a warning to keep your PC plugin.

*BTW, if you lost power during BIOS update, it'll cause permanent and irreversible damage to your PC. You can search it on YouTube, there are videos show you what will happen.*

After few minutes when it finished. Check BIOS version in device manager to verify the version is 1.17. More important, check these in PowerShell(admin):
```
[System.Text.Encoding]::ASCII.GetString((Get-SecureBootUEFI db).bytes) -match 'Microsoft UEFI CA 2023'

[System.Text.Encoding]::ASCII.GetString((Get-SecureBootUEFI db).bytes) -match 'Windows UEFI CA 2023'

[System.Text.Encoding]::ASCII.GetString((Get-SecureBootUEFI KEK).bytes) -match 'Microsoft Corporation KEK 2K CA 2023'
```

You should get `True` for these three. Like:

![](https://github.com/arror403/arror403_note/blob/main/pics/verify_pw.png)

On Linux:
```
mokutil --db | grep -Ei "Microsoft.*UEFI.*CA.*2023|Microsoft.*UEFI.*CA.*2011"
mokutil --kek | grep -Ei "Microsoft.*KEK.*2K.*2023|Microsoft.*KEK.*2011"
```

You'll get:
![](https://github.com/arror403/arror403_note/blob/main/pics/Screenshot_20261011_122743.png)

---

### DNS

From my experience on LineageOS. How [AdAway](https://github.com/AdAway/AdAway) works?

In the world of custom ROM, it's common that people root their devices. Although we can set preferred DNS server in Settings, but as its name "preferred" apps are not necessary to use it. That's why Adguard can't block ads system wide.

With root access, AdAway can force all apps using it. This mechanism is similar to [pi-hole](https://github.com/pi-hole/pi-hole). Indeed, using a phone with AdAway as Wi-Fi extender is essentially the same.

How it works? In AdAway you need to setup host resources. You can find them on [filterlists.com](https://filterlists.com/). It'll extend its contents in `/etc/hosts` (Linux, I'm not sure where Android store it.). On Windows it's in `Windows/System32/drivers/etc/hosts`. 




---

### USB DAC issue


#### Part 1 Too loud

As an audiophile, I use Audioquest's Dragonfly Cobalt(discontinued in 2026). After I changed to openSUSE from Windows(2026/6/3), its output is too loud even when set in the minimum volume. 

Quick fix: Use alsamixer to configure in driver level. But it would need manual set every time.

How to automate it? Use `udev`

```
sudo touch /etc/udev/rules.d/99-usb-dac.rules
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

> Inspired when reading manuals.

Cobalt has a feature that its light will change according to the sampling rate of the file you're playing. 44.1 kHz is green, 48kHz is blue, 88.2kHz is orange, 96kHz is white

How to let Cobalt process it without resampling to 48kHz(default) all the time?

```
sudo touch /home/{your_user_name}/.config/pipewire/pipewire.conf.d/custom-rates.conf
```

Write this:

```
context.properties = {
    default.clock.allowed-rates = [ 44100 48000 88200 96000 ]
}
```

But this will raise another problem, all devices will follow this rule, how to set Cobalt the only that follow this rule?


#### Part 3 Specify device

```
sudo touch /home/{your_user_name}/.config/wireplumber/wireplumber.conf.d/99-dragonfly.conf
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

To verify, by using `pw-top`. It would be helpful for non-dragonfly devices.
For Dragonfly, find sample files with specific sampling rate, the color will change accordingly. Most time it'll be 44.1kHz or 48kHz. You might wondering if it make sound batter? It depends from person to person.


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


## TOEIC


![Average](https://github.com/arror403/arror403_note/blob/main/pics/aver.png)

- **Average = 566**

![Disributaion](https://github.com/arror403/arror403_note/blob/main/pics/dist.png)

- **Percentage of people whose score >= 900 is 4.3%**

![900+](https://github.com/arror403/arror403_note/blob/main/pics/norm_dist_900.png)

- **With the above two, we can calculate standard deviation in normal distribution.**

![815+](https://github.com/arror403/arror403_note/blob/main/pics/norm_dist_815.png)

- **Put my score in. Percentage of people whose score >= 815 is 9.97% (~10%, beat 90%)**


> **Is 90% good? It depends. CEFR C1 requires 945, top 2.5% (2 SD above average)**


---

### Predator Sense issue

[**From my PR**](https://github.com/cleyton1986/predator-sense/pull/56) 

For the issue I encountered:

> When adjusting the keyboard light using Fn + F10, the brightness of the screen will change to 0%. Can be resolved by logging out and logging in. It happened only once after a fresh boot, and then everything worked fine except for Fn + <- (the hotkey to decrease screen brightness).

After a serious amount of troubleshooting, here is what I found:

**From `sudo evtest /dev/input/event0`:**

```bash
/dev/input/event0:      AT Translated Set 2 keyboard

Event: time 1789205461.615337, -------------- SYN_REPORT ------------
Event: time 1789205463.157653, type 4 (EV_MSC), code 4 (MSC_SCAN), value f0
Event: time 1789205463.157653, type 1 (EV_KEY), code 229 (KEY_KBDILLUMDOWN), value 2
Event: time 1789205463.157653, -------------- SYN_REPORT ------------
Event: time 1789205464.607003, type 4 (EV_MSC), code 4 (MSC_SCAN), value ef
Event: time 1789205464.607003, type 1 (EV_KEY), code 224 (KEY_BRIGHTNESSDOWN), value 2
```


**Fn + F10 was mapped to `code 224 (KEY_BRIGHTNESSDOWN)`**


To fix it, create `/etc/udev/hwdb.d/70-keyboard-fix.hwdb` with:
```
evdev:atkbd:dmi:bvn*:bvr*:bd*:svnAcer*:pnPredatorPH315-54:*
 KEYBOARD_KEY_ef=kbdillumup
 KEYBOARD_KEY_f0=kbdillumdown
```

Follow by
```
sudo systemd-hwdb update && sudo udevadm trigger
```

Use `evtest` to verify the change.

```bash
Event: time 1789207119.625433, -------------- SYN_REPORT ------------
Event: time 1789207122.690741, type 4 (EV_MSC), code 4 (MSC_SCAN), value ef
Event: time 1789207122.690741, type 1 (EV_KEY), code 230 (KEY_KBDILLUMUP), value 2
Event: time 1789207122.690741, -------------- SYN_REPORT ------------
Event: time 1789207123.690377, type 4 (EV_MSC), code 4 (MSC_SCAN), value f0
Event: time 1789207123.690377, type 1 (EV_KEY), code 229 (KEY_KBDILLUMDOWN), value 2
```

Now it should work fine.



[60-keyboard.hwdb in systemd](https://github.com/systemd/systemd/blob/main/hwdb.d/60-keyboard.hwdb) includes the following:

```
# Predator PH 315-52
evdev:atkbd:dmi:bvn*:bvr*:bd*:svnAcer*:pnPredator*PH*315-52:*
 KEYBOARD_KEY_ef=kbdillumup                             # Fn+F10
 KEYBOARD_KEY_f0=kbdillumdown                           # Fn+F9
```
