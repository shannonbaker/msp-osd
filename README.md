# IMPORTANT
As of msp-osd v0.12+, the required font format has now changed for the goggles and the OSD Overlay tool.  Support for .bin font file format has been removed in favour of .png font file format.
See the 'Fonts' section and the 'Overlaying OSD on DVR' sections further below for clarification.

# MSP-OSD - Full OSD / Displayport OSD

This package gives you support for full flight controller driven OSD in line with those on analog + other HD systems.

Technically - it takes MSP DisplayPort (so-called "canvas" although this is a misnomer as there is another Betaflight "canvas" mode for Pixel OSDs) messages through UDP and renders them to a framebuffer overlaid under the DJI 'dji_glasses' menu system.

SFML (PC/Mac development) and DJI Goggles viewports are available, as well as a *mux* for the Air Unit / Vista, which creates a *pty* and provides filtered MSP access, and reroutes DisplayPort messages to UDP.

# Setup and Installation

## Easy Installation

* Install WTFOS from https://fpv.wtf. WTFOS must be installed on both the goggles and each AU/Vista.
* Install the msp-osd package on each device using WTFOS.
* Reboot.

## Flight Controller Setup

* Ensure that the correct UART is set to use MSP
* Enable MSP DisplayPort

### Betaflight - Since 4.5

Betaflight 4.5 has added support for coloured warning messages/OSD elements.
This added feature requires a Betaflight font that includes the extra colour elements.  The extra colour elements are for green, amber and red indicators and respectively reside in additional font pages.
This means that to support the extra 3 colours, a 4 page font file is required.

MSP-OSD 0.12+ now supports this feature and includes the required 4 page coloured font for Betaflight bundled by default.
A Betaflight CLI command may needed to take advantage of the this feature:

`set displayport_msp_fonts = 0,1,2,3`

This tells Betaflight to use a specific [colour] page when displaying the warning.

##### *Important*
If you are **not** using a Betaflight 4 page font, and this value is set to `0,1,2,3`, some OSD elements may not display.  In this case, set the value to `0,0,0,0`.  Meaning every warning will use the first page of the font.

### Betaflight - Since 4.4

Betaflight 4.4 has added native support for OSD in HD aspect ratio.

#### Configure With Preset

There is a preset available: "OSD for Fpv.wtf, DJI O3, Avatar HD", once applied you can then skip ahead to configuring in the OSD tab.

#### Configure Manually

Or to configure manually, first enter the following commands in the CLI tab:

```
set osd_displayport_device = MSP
set vcd_video_system = HD
save
```

And then in the Ports tab, select the peripheral "VTX (MSP + DisplayPort)" for the UART your Vista/Air unit is connected to.

![Ports Tab Setting](/docs/img/ports-vtx.png)

Afterwards, you can configure the OSD elements as normal in the OSD tab.

#### Troubleshooting wrong grid size in BF 4.4 Configurator

It is recommended to enable [compressed transmission](#compressed-transmission) with BF 4.4; (now default). It removes/avoids ordering issues between FC/AU/Goggles bootup - the AU has to tell the FC the grid size it supports.

If you don't want to / can't do this - try rebooting your goggles, then reboot your AU.

### Betaflight - 4.3 or Before

We have a configurator preset available - "FPV.WTF MSP-OSD", just be sure to pick the UART your Vista/Air unit is connected to.

#### Or to configure manually

On *Betaflight*, this is done using the following commands in the CLI tab:

```
set osd_displayport_device = MSP
set displayport_msp_serial = <ConfiguratorUART - 1>
set vcd_video_system = PAL
save
```

Eg.: If the Betaflight Configurator says your DJI VTx is attached to UART2, the value for **<ConfiguratorUART - 1>** is **1** - so you would use ```set displayport_msp_serial = 1```.
Test if the value is correct by typing `save` and after the reboot `get displayport_msp_serial` This command should return the value you set it to. If it returns -1 (and that was not the value you set) then the value was not correct.

For Betaflight - ensure you set the Video Format to PAL or Auto in the OSD tab - otherwise you don't get access to the whole OSD area. Note that currently BF Configurator hides these options once you switch to MSP for OSD; the last command above should have done this for you.

#### Softserial

We don't recommend connecting via soft serial; results have been poor - it gives slow/laggy/inconsistent behaviour. But some users have reported it being usable, so if for whatever reason this is your only option, read on.

If you have connected the Vista/Airunit to a softserial port run the `serial` command to list serial ports
Use the value after _serial_ with set `displayport_msp_serial` but do **not** subtract 1 from the value. E.g.:
```
# serial
serial 20 1 115200 57600 0 115200
serial 0 64 115200 57600 0 115200
serial 1 0 115200 57600 0 115200
serial 30 1 115200 57600 0 115200
# set displayport_msp_serial = 30
```

#### FakeHD
With the introduction of HD FPV and its associated transmission and display resolutions, a significantly larger area (canvas*) became a possibility to use for flight controller OSD informaton.
The first generation of the DJI FPV system supports a display resolution of 810p (1440px width x 810px height).  This allowed for a potential for a 60 * 22 grid of (mono-spaced) characters of OSD.
Prior to Betaflight(BF) 4.4, the BF OSD supported a 30 * 16 Grid, which looks large/blocky when displayed in the DJI Goggles.  This 30 * 16 grid was due to the analogue roots of the video and OSD system.

To present the OSD in a visually better way on HD FPV systems, MSP-OSD introduced a workaround called FakeHD that divided up the OSD canvas area into sections that could be configured.

[FakeHD information is available here.](FAKEHD.md)

### INAV

Select "MSP Display Port" (or "HDZero VTx" on older INAV versions) as the Peripheral. Next, select "HD" (or the "WTFOS" variant) in the OSD tab if you'd like to use the HD Canvas.

If the iNav OSD appears garbled at first, try entering the iNav menus using the RC sticks, and then exiting the menus. This will force INAV to switch into HD mode a second time.

It is recommended (now enabled by default in recent msp-osd releases) to enable [compressed transmission](#compressed-transmission) with INAV to avoid issues with the display corrupting - the artifical horizon is the most common element to show this.

### Ardupilot

Please install a recent Ardupilot version for full functionality. There was one critical bug fix for MSP telemetry not passing through a DisplayPort serial port. Additionally, there were several feature additions including HD support after the last 4.2 release.

Settings:

```
SERIALx_PROTOCOL = 42
OSD_TYPE = 5
```

Recent versions of MSP-OSD fully support Ardupilot with a specific HD FPV font.  This font is bundled with msp-osd 0.12
If you wish to use a Betaflight font instead of an Ardupilot font, you can also set `MSP_OPTIONS = 4` to allow the use of a Betaflight font.

More info: https://ardupilot.org/plane/docs/common-msp-osd-overview-4.2.html#dji-goggles-with-wtf-osd-firmware

### KISS Ultra

Select MSP on serial and select DJI WTF as canvas dialect. That's it.

### QUICKSILVER

Configure the UART under Digital VTX - see https://docs.bosshobby.com/Configuring-Quicksilver/#setup

# Fonts

We bundle in default fonts for Betaflight, Ardupilot, INAV, Quicksilver, and KISS ULTRA (INAV/Betaflight/Ardupilot fonts are SNEAKY_FPV's Unify Europa design - thanks to SNEAKYFPV for allowing us to use these - https://sites.google.com/view/sneaky-fpv/home). Since 0.12 we now use a PNG font format, the same as Walksnail. [Default fonts can be viewed here](fonts). You may also upload your own fonts to the SD card.

For the naming convention of the font file in png format see the *FC Specific Font File Names* section below.

***It is important to note the following in regards to when a font file is used from where***

When the goggles determines what font file to use in presenting the OSD, it will look in the following 3 different locations in preferential order:
1. SD Card (root) `/storage/sdcard0`
2. Goggles user font location `/blackbox` (see the *Moving fonts to the goggles file system* section below)
3. Goggles bundled fallback location `/blackbox/wtfos/opt/fonts`

### HD vs SD fonts

Due to historical support for FC firmware that did not have the full HD canvas grid size, both SD and HD font files are bundled.
SD font file/s (naming without the suffix *_hd*) are used when the FC configuration is not set to, or does not support, HD. i.e. BF 4.3 and below.
HD font file/s are used with most modern FC firmware when the equivalent HD selection is made in the FC's OSD configuration tab.

##### *Note*
You do not need to use/copy/install the non-hd font file if you only use a HD OSD canvas.

### Attaining/using fonts
* Download a font package. See below for known community fonts.
* Place these two PNG files on the root of your Goggles SD card.
* Insert the SD card and reboot the goggles.

### Using Walksnail Fonts
As of msp-osd 0.12, the format of font files are now compatible with Walksnail fonts, with the following caveats.

For INAV specific Walksnail fonts, the format of the font pages matter as historical fonts typically were formatted with pages stacked vertically.  The new requirement is for the pages to be stacked side by side. The newer format of side by side pages is supported by both msp-osd and Walksnail.  Side by Side specifially formatted Walksnail fonts for INAV will be denoted 'sbs' in the zipfile and or image names.  This does not affect Betaflight as historical fonts were single page, and newer 4 page fonts are side by side formatted pages.  Ardu is a single page font file so all historically created fonts will work without issue.

A Walksnail font package will typically contain 3 files.  An ini file and 2 png files.  The 2 png files will typically follow a naming convention that contains `_24` or `_36` in the file name.  The '24' file is the msp-osd `_hd` equivalent, and the '36' file is the non-hd equivalent.  e.g.
```
font_update.ini
WS_BFx4_Sphere_24.png
WS_BFx4_Sphere_36.png
```
From the above example, to use them for msp-osd, copy the 2 png files to the SD card root and rename them as:
|Walksnail Name|msp-osd Name|
|--------------|------------|
|`WS_BFx4_Sphere_24.png`|`font_btfl_hd.png`|
|`WS_BFx4_Sphere_36.png`|`font_btfl.png`|

### .bin Fonts

Due to the changes in 0.12, the .bin font format was superseded in favour of a wider accepted format that makes it easier for the community to create their own.

With 0.12, any fonts stored on the SD card in the .bin format will be ignored.

If you wish to retain the font from the .bin file, this open source tool will assist in conversion to png from bin.
https://github.com/shellixyz/hd_fpv_osd_font_tool/tree/main/src/bin/hd_fpv_osd_font_tool

### FC Specific Font File Names

| Flight controller | SD | HD |
| ----------------- | -- | -- |
| Betaflight       | `font_btfl.png` | `font_btfl_hd.png` |
| INAV       | `font_inav.png` | `font_inav_hd.png`|
| Ardupilot       | `font_ardu.png` | `font_ardu_hd.png`|
| KISS Ultra       | `font_ultr.png` | `font_ultr_hd.png`|
| QUICKSILVER       | `font_quic.png` | `font_quic_hd.png`|
| Generic/Fallback*       | `font.png` | `font_hd.png`|

*This uses the Betaflight font layout

Airside VTx (AU/Vista) which have a very old version of msp-osd on, as well as flight controllers which do not respond to the Variant request, like old Ardupilot versions, will use to the Generic/Fallback font.
##### *Note*
You can also add fonts for firmwares not in this list; using the generic filename, or put the MSP identifier in (lower case it) the filename - ```font_<fc_variant>.png / font_<fc_variant>_hd.png```

### Moving fonts to the goggles file system

If you wish to use a specific font different than the font stored on the goggles, for when you do not have the SD card inserted in the goggles, you can follow the below steps to do this.
1. Copy the font files you wish to be stored on the goggles firstly to your SD card
2. Connect your goggles via USB if you didn't do this for step 1.
3. Insert the SD card in the goggles then power them on
4. Navigate to https://fpv.wtf
5. After the goggles are found and connected, go to the fpv.wtf CLI and type the following
6. `cp /storage/sdcard0/font*.png /blackbox`
7. Power off the goggles, remove the SD card and confirm the newly copied font is used.

##### *Note*
To remove copied fonts repeat steps 1 to 6 and replace step 7 with `rm /blackbox/font*.png`.  Then power off and on the goggles.

### Suggested Third Party Fonts

 - [KNIFA's Material](https://github.com/Knifa/material-osd/releases) - use the Walksnail version for MSP-OSD <= 0.12
 - [SNEAKY_FPV's colour fonts for INAV, ARDU and BF](https://sites.google.com/view/sneaky-fpv/home)
 - [VICEWIZE Italic](https://github.com/vicewize/vicewizeosdfontset)
 - [Kw0ngk4n's Neue OSD](https://github.com/Kw0ngk4n/WTF-Neue-OSD)
 - [EVilm1's OSD Font](https://github.com/EVilm1/EVilm1-OSD-Font)


## Overlaying OSD on DVR
**Note the change in font file format requirements (png format) for the OSD Overlay tool.  If no font file is provided by the user, a built in font (png format) will be used.**

The overlay process uses DVR, recorded osd data and a font file, to overlay/render the OSD data onto DVR footage on your computer.
A default font will be used if no font file is supplied however.  The font look and feel will be the bundled font you would see if using this during flight.

https://fpv.wtf/osd-overlay provides a tool that will overlay captured osd information onto DVR footage.
### Pre-requisites
Unless the osd information is captured during DVR recording on the goggles you will be unable to overlay your osd onto DVR on your computer.

The following fpv.wtf CLI commands (goggles) will ensure this is enabled.

```
package-config set msp-osd rec_enabled true
package-config apply msp-osd
```

##### *Notes*
You only need to supply a font file for the canvas the DVR was recorded with.  i.e. If your FC firmware configuration was HD or a HD variant you only need to supply the '_hd.png' font file.
The 'Chroma Key' will replace the DVR with a solid colour for use within video editing software.  Be aware however that fonts have an amount of transparency around elements that will include the 'Chroma Key' bleed that will be difficult to avoid in video editing software.

See the 'Using Walksnail Fonts Section' for specific requirements that may apply to font file formats.

# Configuration options

Configuration options can be set using the WTFOS Configurator CLI.

Prefix option with `package-config set msp-osd`

e.g. `package-config set msp-osd compress_osd true`

Once desired setting changes are made then: `package-config apply msp-osd`, otherwise settings will be lost when power is off.

Visit https://fpv.wtf/package/fpv-wtf/msp-osd with your Goggles or Air Unit plugged in to edit options.

### Current available options (Goggles):

| Option | Description | Type | Default|
| ------ | ----------- | ---- |--------|
|`show_waiting`| enables or disables WAITING FOR OSD message | true/false | true |
|`show_au_data`| enables AU data (temp/voltage) overlay on the right | true/false | false |
|`rec_enabled`| enable OSD recording to .msp files alongside video | true/false | true |
|`rec_pb_enabled`| enable OSD playback if .msp file is stored alongside video | true/false | true |
|`hide_diagnostics`| hide the diagnostic information in the bottom right | true/false | false |
|`fakehd_enable`| enables [FakeHD](#FakeHD); the other FakeHD options don't do anything if this is disabled. FakeHD is force disabled if the Flight Controller supports proper HD / RealHD | true/false| false |
|`fakehd_lock_center`| Lock FakeHD in centered mode all the time; no gaps/spreading out even when you are flying. | true/false | false |
|`fakehd_menu_switch`| FakeHD will use this character as the menu switch to detect when you are in menus/postflight and triggger centering. | integer/number | 4 (Betaflight Throttle) |
|`fakehd_hide_menu_switch`| FakeHD will hide the menu switch set above; and the next 5 characters | true / false | false |
| `fakehd_columns` | FakeHD column alignment config | Single character, one of T M B S | S |
| `fakehd_rows` | FakeHD row alignment config, each character configures the alignment for one row | 16 characters, each one of L C R W T F D | WWWWWWCCWWWWWWWD |



### Current available options (Air Unit/Vista):

| Option | Description | Type | Default|
| ------ | ----------- | ---- |--------|
|`compress_osd`| Enable sending full frames of compressed data. Disable to send raw MSP data [Read more](#Compressed-Transmission) | true/false| true |
| `osd_update_rate_hz` | Configure the update rate in hz for the OSD when using compressed transmission | integer | 10 |
| `disable_betaflight_hd` | Disable HD Mode, which is otherwise set by default in Betaflight 4.4 | true/false | false |
| `fast_serial` | Change serial baud rate to 230400 baud, which can improve OSD performance in some situations - FC UART config must be changed to match. | true/false | false |
| `cache_serial` | Cache unimportant MSP messages for seldom-used features (like PID tuning in the DJI Goggles Settings Menu) to reduce serial pressure | true/false | false |

#### Compressed Transmission

As of 0.7.0, a new option, `compress_osd`, was added to the **air side** process.

If this is set to "true", then the entire character buffer will be sent using LZ4 compression at the rate defined in `osd_update_rate_hz`, instead of sending raw MSP messages over the air.

When enabled, this should fix INAV delta update related issues as well as provide better link stability.

To enable:

Visit https://fpv.wtf/package/fpv-wtf/msp-osd with your **Air Unit / Vista** plugged in to edit package options.

This option is enabled by default as of 0.10.0, however, if you upgraded from an older version, your configuration will need to be updated using the configurator.

If you continue to have issues with especially INAV character corruption, it is likely your serial link is saturated. Check that the "Custom OSD" option in your DJI goggles menus is set to _disabled_ , and also try out the cache_serial option.

## FAQ / Suggestions

### Modify / Move original DJI OSD elements

You can now modify the elements present in the original DJI OSD. These include for example : transmission speed, latency, channel used, googles battery, sd card icon and default timer.

Elements position, visibility, font and icons can be modified by editing the internal googles files.
This is possible by connecting to the googles using ADB. You can even preview changes using a Python script!

This is not a trivial thing for everyone to do, the full tutorial can be found [here](https://github.com/EVilm1/WIKI-HACK-DJI-OSD#6-advanced-setup-modify-the-dji-hud-elements).

### How do I create a new font (for INAV, Ardupilot, etc.)?

Use `mcm2img` , specifically Knifa's branch to allow you to draw using a PNG template.

https://github.com/Knifa/mcm2img/tree/templates

### Why is everything so big / can I make the text smaller (betaflight)?

For Betaflight prior to 4.4, look into FakeHD.
For Betaflight after 4.4, you should see "HD" fonts by default. Make sure your VTx (AU/Vista) is powered up and visit the Betaflight Configurator to move OSD items to the edge of the screen.

### How can I get my INAV/ArduPilot/Kiss Ultra OSD closer to the edge of the screen / Why is FakeHD closer to the edges?

 - The goggles need 60 characters to go edge to edge - so the 50 in the hd grid doesn't quite fill it
 - So, depending on the Flight Controller's setup, the RealHD grid is displayed centered in the goggles - gaps on both edges.
 - FakeHD had no compatibility constraints like this so we were able to use the full width of the screens.
 - Consequently, FakeHD can get nearer the edges.
 - Currently no solution to get RealHD closer to the edges.

### What is RealHD

Sometimes we refer to the proper MSP OSD HD grid supported by ArduPilot / Kiss Ultra / INAV / Betaflight (from 4.4) + others as RealHD, to distinguish from FakeHD.

# Compiling (development and debugging)

To build for DJI, install the [Android NDK](https://developer.android.com/ndk/downloads) and add the NDK toolchain to your PATH, then use `ndk-build` to build the targets.

To build for UNIXes, install CSFML and run:

```
make -f Makefile.unix
```

Provided targets and tools are:

* `msp_displayport_mux` - takes MSP DisplayPort messages, bundles each frame (all DisplayPort messages between Draw commands) into a single UDP Datagram, and then blasts it over UDP. Also creates a PTY that passes through all _other_ MSP messages, for `dji_hdvt_uav` to connect to.
* `libdisplayport_osd_shim.so` - Patches the `dji_glasses` process to listen for these MSP DisplayPort messages over UDP, and blits them to a DJI framebuffer screen using the DJI framebuffer HAL `libduml_hal` access library, and a converted Betaflight font stored in `font.bin`.
* `osd_sfml` - The same thing as `osd_dji`, but for a desktop PC using SFML and `bold.png`.

Additional debugging can be enabled using `-DDEBUG` as a CFLAG.

## Custom Build Installation (Goggles)

There's a slightly different process for V1 vs V2 Goggles, they renamed some bits between the two.

### FPV Goggles V1

```
ndk-build
adb shell setprop dji.dji_glasses_service 0
adb push libs/armeabi-v7a/libdisplayport_osd_shim.so /tmp
adb shell LD_PRELOAD=/tmp/libdisplayport_osd_shim.so dji_glasses_original -g
```

### FPV Goggles V2

```
ndk-build
adb shell setprop dji.glasses_wm150_service 0
adb push libs/armeabi-v7a/libdisplayport_osd_shim.so /tmp
adb shell LD_PRELOAD=/tmp/libdisplayport_osd_shim.so dji_gls_wm150_original -g
```

### Air Unit / Air Unit Lite (Vista)

```
ndk-build
adb push msp_displayport_mux /blackbox
adb shell setprop dji.hdvt_uav_service 0
adb shell mv /dev/ttyS1 /dev/ttyS1_moved
adb shell nohup /blackbox/msp_displayport_mux 192.168.41.2 /dev/ttyS1_moved /dev/ttyS1 &
adb shell setprop dji.hdvt_uav_service 1
```

This tells the displayport mux to send data from /dev/ttyS1_moved to 192.168.41.2 (goggles) and to create a fake serial port at /dev/ttyS1 with the displayport messages filtered out.

Optionally, you can add `-f`, like `nohup /blackbox/msp_displayport_mux -f 192.168.41.2 /dev/ttyS1_moved /dev/ttyS1` to put the serial port in a faster 230400 baud mode, and set the MSP serial port in your flight controller to 230400 to try to improve the framerate.

You can also omit `setprop dji.hdvt_uav_service 1` , which will improve your OSD framerate at the expense of disabling all Air Unit / Vista side coordination functionality (AU recording, channel changes, some RC features, etc.).

Enjoy.

## Additional Reading / Learning

https://github.com/fpv-wtf/margerine/wiki

## Shoutouts / Thank You

* http://github.com/fpv-wtf team, for making this all possible and very helpful random bits of advice
