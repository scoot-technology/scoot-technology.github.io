---
layout: post
title:  "Turntable Integration With Snapcast"
date:   2023-04-02
categories: [Computing, Hi-Fi, Snapcast]
feature_image: assets/snapcast/snapcast_turntable.jpg
excerpt: Ever wanted your turntable, CD player or other physical audio source integrated into your multi-room audio setup? In this post we look at doing just that with Snapcast.
---

 > If you haven't already be sure to read my [Introduction to Snapcast](https://scoot-technology.github.io/posts/2023-02-26-multi-room-audio-with-snapcast.html) blog post

 > This post will focus on Linux. It is possible to stream via TCP to Snapcast from a Windows PC but I won't be covering that here.

So you've decided to give Snapcast a go and maybe by now you have some [sources integrated](https://github.com/badaix/snapcast/blob/develop/doc/player_setup.md) like [Mopidy](https://mopidy.com/) or [Music Player Daemon](https://www.musicpd.org/). But What about that CD player or turntable confined to the Hi-Fi rack in the corner of the lounge? In this post I will discuss some options of how you can integrate these sources into your [Snapcast](https://github.com/badaix/snapcast) multi-room setup for synchronised playback all around the house.

# What Do I Need?
First of all let me make it clear that you will either need your Snapcast computer in close proximity to the Hi-Fi component you want to integrate or you will need another computer nearby which can stream your physical source over the network. Once you have worked that bit out the next thing to determine is the best way to capture your source which generally will be one of the following 3 (in order of preference):

 * **USB output:** If your device has a digitisation feature and provides a USB output such as many modern turntables, phono preamps and cassette players this will be the simplest method of capturing it's audio on your computer in order to stream with Snapcast.

 * **Digital Output:** If your device has a digital output such as SPDIF you can use an SPDIF to USB converter to capture the audio such as [this one from Hifime](https://hifimediy.com/product/hifime-ur23-spdif-optical-to-usb-converter/). The advantage of using a SPDIF output with digital sources such as streamers, CD or MiniDisc players is it avoids an analogue conversion in the chain meaning Snapcast can capture and stream the sources audio more or less direct. There are caveats to this which I will cover later in the post.

 * **Analogue Outputs:** If all else fails or your source is analogue you can use an analogue capture device to digitise the signal coming from your source so it can be streamed with Snapcast. This step will of course involve an analogue to digital conversion so results will vary depending on the quality of your capture device. Many computer sound cards have a 3.5mm line input which could be used to capture your analogue source or for even better results a higher quality RCA to USB capture device or external audio interface could be used. Just be sure to check that Linux is supported.

 > **Digital Outputs:** Before going out and purchasing a digital capture device you need to understand the properties of your source with respect to sample rates in particular. Sources such as CD players tend to output only 44.1 KHz sample rates so a basic capture device like [this one from Hifime](https://hifimediy.com/product/hifime-ur23-spdif-optical-to-usb-converter/) will work well. If your source is likely to output variable sample rates such as a DVD player or Chromecast Audio you will ideally require a capture device with a DSP capable of sample rate conversion. I personally use a [HiFiBerry Dac+ DSP](https://www.hifiberry.com/shop/boards/hifiberry-dac-dsp/) for capturing a Chromecast Audio which I will cover in a future post. If on the other hand you know the output of your source will be a constant sample rate due to the media you will be playing or the sources ability to resample the audio internally you don't need to worry.

# Getting Started

Regardless of whether you are going to capture using a USB, digital or analogue output the principle for getting the source into your Snapserver is the same. There are two ways I recommend to do this:

* **ALSA Capture:** This is the best option if you are capturing on the same computer that is hosting your Snapserver as you get [native ALSA support](https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#alsa) to make things simple.

* **TCP Streaming:** TCP streaming can be used if you will be using a different computer to capture your source than the one hosting your Snapcast Server. [Snapcast's TCP implementation](https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#tcp-server) is an easy way to stream audio sources over a standard wired or wireless network.

Assuming you have already sorted the hardware required to capture your source and it is Linux compatible the next step is to determine the name of your device when connected to your computer. The easiest way to do this is to use a useful ALSA tool called `arecord`. This is part of the `alsa-utils` package on most distributions.

 > **Note:** Your user will need to be added to the `audio` group in Linux for `arecord` or any other ALSA based commands to work without elevated privileges.

As an example here is the output of `arecord -L` with my Audio Technica LP5x turntable connected via USB:

    arecord -L

    null
        Discard all samples (playback) or generate zero samples (capture)
    hw:CARD=CODEC,DEV=0
        USB AUDIO  CODEC, USB Audio
        Direct hardware device without any conversions
    plughw:CARD=CODEC,DEV=0
        USB AUDIO  CODEC, USB Audio
        Hardware device with all software conversions
    default:CARD=CODEC
        USB AUDIO  CODEC, USB Audio
        Default Audio Device
    sysdefault:CARD=CODEC
        USB AUDIO  CODEC, USB Audio
        Default Audio Device
    front:CARD=CODEC,DEV=0
        USB AUDIO  CODEC, USB Audio
        Front output / input
    dsnoop:CARD=CODEC,DEV=0
        USB AUDIO  CODEC, USB Audio
        Direct sample snooping device

In this instance there is only one capture device present which can be identified by `CARD=CODEC`. There are many sub-devices which ALSA makes available but generally I only want to work with the device direct which in this case is identified as `hw:CARD=CODEC,DEV=0`. Make a note of the relevant direct device identifier for your setup which will be used in subsequent sections.

# ALSA Capture (Option 1)

As previously mentioned Snapcast has [native ALSA support](https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#alsa) which can be used to capture your source if it is in close proximity to the computer hosting your Snapcast server. First of all it's important to understand that Snapcast uses a global sample format for all sources and clients which is defined in your Snapserver config:

    sampleformat = 48000:16:2

Default is 48KHz as above but if you intend to use Spotify or AirPlay as a source or mostly listen to CD quality sources you will want to change this to 44.1KHz:

    sampleformat = 44100:16:2

Snapcast currently has no resampling capability so if your ALSA capture device is unable to provide audio in the sample format defined in your Snapserver config you will need to move on to Option 2. If ALSA can open your capture device at the desired sample format then you are good to add a source like the example below to your Snapserver config:

    source = alsa:///?name=Phono&device=hw:CARD=CODEC,DEV=0&idle_threshold=100&silence_threshold_percent=0.5

You can find more information on the source parameters [here](https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#alsa) but below is a brief overview:
 * **name :** This is whatever you want the source to appear as in the web interface and other control interfaces.
 * **device :** This is the ALSA device identifier we noted previously.
 * **idle_threshold** : How many milliseconds of silence before the source transitions from *Playing* to *Idle*.
 * **silence_threshold_percent :** The percentage of max amplitude to be considered silence. For digital sources this can probably be *0.0*. For analogue sources with varying noise floors you may need to adjust this accordingly.

Restart your Snapserver instance and all being well your new source will be available to assign to the client(s) of your choice.

# ALSA Capture (Option 2)

If you choose to use this option it's likely that it's because your source requires resampling to work with Snapcast. There are several ways you can do this but the most robust option I have found is to use [GStreamer](https://gstreamer.freedesktop.org/) together with a FIFO pipe.

First things first we will add a [**pipe source**](https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#pipe) to our Snapserver config:

    source = pipe:///tmp/phonofifo?name=Phono&mode=read

After restarting our Snapserver we will get a new source available called `Phono` which will read raw audio data from a pipe located at `/tmp/phonofifo`. Of course this doesn't yet exist so lets fix that by using GStreamer to handle our ALSA capture device, resample the audio and send the data to our pipe. I would recommend you do this using [Docker](https://www.docker.com/) or even better [Docker Compose](https://docs.docker.com/compose/) but you can also use GStreamer natively if you have it installed.

**Native**

    gst-launch-1.0 alsasrc device=hw:CARD=CODEC,DEV=0 ! audioconvert ! removesilence remove=true ! audioconvert ! audioresample ! audio/x-raw,format=S16LE,channels=2,rate=44100 ! filesink location=/tmp/phonofifo

**Docker**

    docker run -d -v /tmp:/tmp --device /dev/snd --name snapcast_phono scootsoftware/gstreamer alsasrc device=hw:CARD=CODEC,DEV=0 ! audioconvert ! removesilence remove=true ! audioconvert ! audioresample ! audio/x-raw,format=S16LE,channels=2,rate=44100 ! filesink location=/tmp/phonofifo

**Docker Compose**

    version: '3'
    services:
    snapcast_phono:
        container_name: snapcast_phono
        image: scootsoftware/gstreamer
        restart: unless-stopped
        volumes:
          - /tmp:/tmp
        devices:
          - /dev/snd:/dev/snd
        command: alsasrc device=hw:CARD=CODEC,DEV=0 ! audioconvert ! removesilence remove=true ! audioconvert ! audioresample ! audio/x-raw,format=S16LE,channels=2,rate=44100 ! filesink location=/tmp/phonofifo

Lets break down the command:

 * `alsasrc device=hw:CARD=CODEC,DEV=0`: This is our source which is an ALSA capture device so we use the identifier we noted previously.
 * `removesilence remove=true`: We use the `removesilence` plugin to detect when no audio is playing so we can stop sending data to Snapcast and the source status will report correctly. If you are using a digital capture device which only provides data when a signal is detected this can be omitted but if you find your source always reports *Playing* this will fix it.
 * `audioresample ! audio/x-raw,format=S16LE,channels=2,rate=44100`: This ensures the captured audio is resampled before reaching Snapcast. In this case we convert and resample to a 44.1KHz/16-bit output but change this as required depending on the global format in your Snapserver config.
 * `filesink location=/tmp/phonofifo`: This is our output which is a pipe at the location `/tmp/phonofifo`.

# TCP Streaming

TCP streaming is a way of sending audio data over your network which is convenient if your source isn't located near to your Snapserver host. You will of course need another computer located near to your source. It's worth noting that TCP streaming can also be used as an alternative to *Option 2* above if you would prefer not to use a pipe for any reason.

Firstly we will add a [**TCP server source**](https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#tcp-server) to our Snapserver config:

    source = tcp://0.0.0.0:4900?name=Phono

In this example we setup a TCP server listening on port `4900`. The port can be whatever you want as long as it is not already in use. If you have multiple TCP server sources in your Snapserver config the port needs to be different for each of them. After restarting our Snapserver we will get a new source available called `Phono`.

As with *Option 2* I prefer using GStreamer to handle our ALSA capture device, resample the audio and stream the output to our TCP server. Below are the commands to do this.

**Native**

    gst-launch-1.0 alsasrc device=hw:CARD=CODEC,DEV=0 ! audioconvert ! removesilence remove=true ! audioconvert ! audioresample ! audio/x-raw,format=S16LE,channels=2,rate=44100 ! tcpclientsink host=my-snapserver port=4900

**Docker**

    docker run -d -v /tmp:/tmp --device /dev/snd --name snapcast_phono scootsoftware/gstreamer alsasrc device=hw:CARD=CODEC,DEV=0 ! audioconvert ! removesilence remove=true ! audioconvert ! audioresample ! audio/x-raw,format=S16LE,channels=2,rate=44100 ! tcpclientsink host=my-snapserver port=4900

**Docker Compose**

    version: '3'
    services:
    snapcast_phono:
        container_name: snapcast_phono
        image: scootsoftware/gstreamer
        restart: unless-stopped
        volumes:
          - /tmp:/tmp
        devices:
          - /dev/snd:/dev/snd
        command: alsasrc device=hw:CARD=CODEC,DEV=0 ! audioconvert ! removesilence remove=true ! audioconvert ! audioresample ! audio/x-raw,format=S16LE,channels=2,rate=44100 ! tcpclientsink host=my-snapserver port=4900

In the above examples you will need to make sure you specify the correct ALSA `device` which we noted previously. You will also need to specify your `host` which is the hostname or IP address of the computer hosting your Snapserver instance. Finally make sure `port` matches what you configured in your Snapserver source.

# Conclusion
Hopefully this post has helped you to understand how you might be able to integrate many different types of sources into your multi-room setup in order to enjoy your favorite music, in whatever format, wherever you choose. Snapcast gives us the freedom to be creative with the sources available in our multi-room setup rather than being limited to specific devices with compatibility for proprietary solutions like *Works With Sonos*.

In the next post in the series I will go into more details on how I integrate digital sources with inconsistent sample formats such as the Chromecast Audio. See you there!