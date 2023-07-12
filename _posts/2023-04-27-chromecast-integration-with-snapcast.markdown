---
layout: post
title:  "Chromecast Integration With Snapcast"
date:   2023-04-27
categories: [Computing, Hi-Fi, Snapcast]
feature_image: assets/snapcast/snapcast_chromecast_audio.jpg
excerpt: When Google retired the Chromecast Audio it was the catalyst for me to try an open-source multi-room solution like Snapcast. But what if you still want the ability to cast from your favourite apps? This post explores one way you can do this.
---

 > If you haven't already be sure to read my [Introduction to Snapcast](https://scoot-technology.github.io/posts/2023-02-26-multi-room-audio-with-snapcast.html) and [Snapcast Source Integration](https://scoot-technology.github.io/posts/2023-04-02-turntable-integration-with-snapcast.html) posts.

If you are moving to [Snapcast](https://github.com/badaix/snapcast) from an ecosystem like Google Cast losing the ability to cast from your favourite apps can be a deal breaker. Whilst there are great options for integrating [Spotify](https://github.com/librespot-org/librespot) and [AirPlay](https://github.com/mikebrady/shairport-sync) with Snapcast I am yet to come across an open-source Chromecast implementation which means continuing to benefit from the convenience and rich ecosystem of Google Cast isn't so easy. Given I'm not aware of any pure software based Chromecast implementations the only way to add Chromecast support to your Snapcast setup is to... add a Chromecast.

I mentioned in [this post](https://scoot-technology.github.io/posts/2023-04-02-turntable-integration-with-snapcast.html) several different ways you can integrate physical audio sources and HiFi components into your Snapcast setup depending on the audio outputs available. In my case I had replaced a collection of Chromecast Audio devices with various Snapcast clients so I wanted to integrate one of these Chromecast Audios into my Snapcast setup but of course any Chromecast capable streamer with a suitable audio output is an option.

# What's The Problem?
Unlike many digital sources such as CD players or MiniDisc players, streamers like the Chromecast Audio are capable of outputting audio at different sample rates and bit depths depending on the source stream. Generally with a CD player you know that the audio coming out of the digital output is going to be constant (e.g. 44.1KHz, 16-bit). With a streamer the audio sample format is likely to be different depending on whether you are streaming a radio broadcast or a high resolution track from your favourite streaming service. This is a problem because basic digital capture devices such as [this one from Hifime](https://hifimediy.com/product/hifime-ur23-spdif-optical-to-usb-converter/) have no mechanism to allow automatic switching of sample formats to dynamically resample the audio in software and there is no hardware based mechanism to achieve this. If you start capturing at 48KHz and then play a high resolution 96KHz stream the resulting audio will be garbled.

You may be thinking... why not just use the analogue output to capture the audio? In honesty this is a perfectly viable option but the purist in me has a couple of issues compared to capturing a digital signal.

1. In this scenario you are using the internal DAC of whatever device you are capturing which will colour the signal before being streamed to your Snapcast clients. If, like me, you have dedicated DACs connected to your Snapcast clients which are significantly better than the one in the source you are capturing the resulting audio quality is likely to disappoint.

2. Capturing an analogue signal as apposed to digital means an additional analogue to digital conversion in the chain. The quality of the capture device performing the conversion will also have a significant impact on the resulting audio quality sent to your Snapcast clients.

If the source is digital I prefer to keep everything in the digital domain for as long as possible to avoid unnecessary conversions and keep the audio streamed to the client as true to the source as possible. With my Chromecast Audio that means capturing the audio from the optical output.

# What Do I Need?
To deal with the variable sample formats the Chromecast Audio is capable of transmitting we need some hardware in the capture chain which can resample everything to a constant format. One solution is a Digital Signal Processor (DSP) which is able to resample the digital input to a constant format amongst other things. One option is the [HiFiBerry DAC+ DSP](https://www.hifiberry.com/shop/boards/hifiberry-dac-dsp/) which is compatible with Raspberry Pi 2/3/4 and incorporates an optical input, optical output, DAC and DSP on a single board. If you already use a Raspberry Pi for your Snapcast server this is a great option. There is also a [DSP addon board](https://www.hifiberry.com/shop/boards/dsp-add-on/) for several other HifiBerry boards.

<p align="center">
    <img src='{{ "assets/hifiberry/hifiberry_dac_plus_dsp.jpg" | relative_url }}' height="500">
</p>

# DAC+ DSP Setup
The HifiBerry DAC+ DSP offers a lot of functionality but for the purposes of this post I will focus on capturing audio from the optical input and streaming with Snapcast. This involves routing the DSP output back to the Raspberry Pi so it can be captured with ALSA. By default the DSP output is routed to the analogue RCA outputs and/or optical output of the DAC+ DSP board so it can work as a standalone DAC.

To route the DSP audio output back to the Raspberry Pi there are a couple of steps required as [documented here](https://www.hifiberry.com/docs/software/using-the-dac-dsp-to-record-audio-from-s-pdif/).

Firstly we need to use a different driver than what is specified in the default setup guide for the DAC+ DSP in `/boot/config.txt`:

    dtoverlay=hifiberry-dacplusdsp

After a reboot you can use the `arecord -L` command to check the DAC+ DSP is available to capture:

    null
        Discard all samples (playback) or generate zero samples (capture)
    hw:CARD=sndrpihifiberry,DEV=0
        snd_rpi_hifiberrydacplusdsp_sou, Hifiberry DAC+DSP SoundCard HiFi dacplusdsp-hifi-0
        Direct hardware device without any conversions
    plughw:CARD=sndrpihifiberry,DEV=0
        snd_rpi_hifiberrydacplusdsp_sou, Hifiberry DAC+DSP SoundCard HiFi dacplusdsp-hifi-0
        Hardware device with all software conversions
    default:CARD=sndrpihifiberry
        snd_rpi_hifiberrydacplusdsp_sou, Hifiberry DAC+DSP SoundCard HiFi dacplusdsp-hifi-0
        Default Audio Device
    sysdefault:CARD=sndrpihifiberry
        snd_rpi_hifiberrydacplusdsp_sou, Hifiberry DAC+DSP SoundCard HiFi dacplusdsp-hifi-0
        Default Audio Device
    dsnoop:CARD=sndrpihifiberry,DEV=0
        snd_rpi_hifiberrydacplusdsp_sou, Hifiberry DAC+DSP SoundCard HiFi dacplusdsp-hifi-0
        Direct sample snooping device

The device we will use in the subsequent steps to capture audio in this instance is `hw:CARD=sndrpihifiberry,DEV=0`.

Next we need to configure the DSP on the DAC+ DSP board to route the optical input to the Asynchronous Sample Rate Converter (ASRC) and from there onto the Raspberry Pi. Execute the following commands to get this configured:

    dsptoolkit write-reg 0xF106 0x0003
    dsptoolkit write-reg 0xF146 0x0004
    dsptoolkit write-reg 0xF195 0x0000
    dsptoolkit write-reg 0xF194 0x0033
    dsptoolkit write-reg 0xF21C 0x6C40

At this point any audio signal detected on the optical input of the DAC+ DSP board will be resampled and routed to the capture device we noted previously.

# Snapcast Setup
There are two main options available for capturing the DAC+ DSP audio with Snapcast. Which one you choose will largely depend on whether your Snapcast server is running on the same Raspberry Pi as the DAC+ DSP or a separate computer.

* **ALSA Capture:** This is the best option if you are capturing on the same computer that is hosting your Snapcast server as you get [native ALSA support](https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#alsa) to make things simple.

* **TCP Streaming:** TCP streaming can be used if your Snapcast server is hosted on a different computer to the Raspberry Pi with DAC+ DSP. [Snapcast's TCP implementation](https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#tcp-server) is an easy way to stream audio over a standard wired or wireless network.

**ALSA Capture**

As previously mentioned Snapcast has [native ALSA support](https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#alsa) which can be used to directly capture the audio from the DAC+ DSP. Add a source like the example below to your Snapserver config:

    source = alsa:///?name=Chromecast&device=hw:CARD=sndrpihifiberry,DEV=0&idle_threshold=100

You can find more information on the source parameters [here](https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#alsa) but below is a brief overview:
 * **name :** This is whatever you want the source to appear as in the web interface and other control interfaces.
 * **device :** This is the ALSA device identifier we noted previously.
 * **idle_threshold** : How many milliseconds of silence before the source transitions from *Playing* to *Idle*.

Restart your Snapserver instance and all being well your new source will be available to stream to the client(s) of your choice.

**TCP Streaming**

TCP streaming is a way of sending audio data over your network which is convenient if your Raspberry Pi with DAC+ DSP isn't located near to your Snapcast server host. It's worth noting that TCP streaming can also be used as an alternative to *ALSA Capture* if you encounter issues.

Firstly we will add a [**TCP server source**](https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#tcp-server) to our Snapserver config:

    source = tcp://0.0.0.0:4900?name=Chromecast

In this example we setup a TCP server listening on port `4900`. The port can be whatever you want as long as it is not already in use. If you have multiple TCP server sources in your Snapserver config the port needs to be different for each of them. After restarting our Snapserver we will get a new source available called `Chromecast`.

I prefer using GStreamer to handle an ALSA capture device and stream the output to a TCP server. Below are the commands to do this depending on your preferred method.

**Native**

    gst-launch-1.0 alsasrc device=hw:CARD=sndrpihifiberry,DEV=0 ! audioconvert ! removesilence remove=true ! audioconvert ! audioresample ! audio/x-raw,format=S16LE,channels=2,rate=48000 ! tcpclientsink host=my-snapserver port=4900

**Docker**

    docker run -d -v /tmp:/tmp --device /dev/snd --name snapcast_chromecast scootsoftware/gstreamer alsasrc device=hw:CARD=sndrpihifiberry,DEV=0 ! audioconvert ! removesilence remove=true ! audioconvert ! audioresample ! audio/x-raw,format=S16LE,channels=2,rate=48000 ! tcpclientsink host=my-snapserver port=4900

**Docker Compose**

    version: '3'
    services:
    snapcast_phono:
        container_name: snapcast_chromecast
        image: scootsoftware/gstreamer
        restart: unless-stopped
        volumes:
          - /tmp:/tmp
        devices:
          - /dev/snd:/dev/snd
        command: alsasrc device=hw:CARD=sndrpihifiberry,DEV=0 ! audioconvert ! removesilence remove=true ! audioconvert ! audioresample ! audio/x-raw,format=S16LE,channels=2,rate=48000 ! tcpclientsink host=my-snapserver port=4900

In the above examples you will need to make sure you specify the correct ALSA `device` which we noted previously. You will also need to specify your `host` which is the hostname or IP address of the computer hosting your Snapserver instance. The `port` needs to match what you configured in your Snapserver source in the previous step. Finally the audio sample format specified needs to match the global sample format defined in your Snapserver config such as `sampleformat = 48000:16:2`. In this instance the sample rate is set to 48 KHz but if you are using a different sample format such as 44.1 KHz you will need to change the above commands accordingly (e.g. `format=S16LE,channels=2,rate=44100`)

# Caveats

At this point you should have a Chromecast device on your network which your Snapcast server can stream to clients around the home. There are a few caveats and downsides to mention which I will list below.

**Casting audio to the right room(s) becomes a 2 step process.** If you have a Google smart speaker or Chromecast Audio in each room casting is as simple as selecting the corresponding speaker(s) from your streaming app and everything is controlled from one place. With this approach you first cast to the relevant Chromecast device and then you need to select the correct source on the Snapcast clients you want to stream to from a separate app or control interface. It's worth mentioning that creating a [meta source](https://scoot-technology.github.io/posts/2023-03-01-snapcast-automatic-source-selection.html) can help with this as it will allow your Snapcast clients to automatically determine which source to stream at any given time based on state and priority.

**Limited integration with Google Assistant.** If you use Google Assistants to get music playing in the correct room or control your speakers there will be some loss of functionality. It is of course possible to set the Chromecast device connected to Snapcast as the default speaker or specifically ask your Google Assistant to play music on that device however you can't ask your assistant to play music in the kitchen for example. It's also worth mentioning that if you use [Home Assistant's integration with Google Assistant](https://www.home-assistant.io/integrations/google_assistant/) and the [Snapcast integration](https://www.home-assistant.io/integrations/snapcast/) you can control your Snapcast clients source and volume however it will be a two command process.

To change the ***Default music speaker*** so that your Google Assistant defaults to playing music on the Chromecast device hooked up to your Snapcast server you can use the Google Home app on your smartphone. Open the app and select the Assistant device(s) you wish to change. Click the `Settings` icon and select `Audio -> Default music speaker` and select the correct Chromecast device from the list.

<p align="center">
    <img src='{{ "assets/google_home/google_home_default_speaker_setting.jpg" | relative_url }}' height="500">
</p>

# Conclusion
Whilst it's definitely a bonus to break free from closed ecosystems like Google Cast it's undoubtedly convenient to still have it as an option. Integrating a Chromecast device as a source for your Snapcast based multi-room setup gives you the ability to cast music or audio from your favourite apps and have it streamed around the home without being limited to Google Cast enabled speakers and devices. Admittedly it's not the most elegant of solutions but it's nice to have a multi-room setup with the ability to support Google Cast alongside the likes of AirPlay and Spotify Connect.

Stay tuned for more [Snapcast](https://mjaggard.github.io/snapcast/) related posts in the near future!