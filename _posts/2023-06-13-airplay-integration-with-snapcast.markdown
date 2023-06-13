---
layout: post
title:  "AirPlay Integration With Snapcast"
date:   2023-06-13
categories: [Computing, Hi-Fi, Snapcast]
feature_image: assets/snapcast/snapcast_airplay.jpg
excerpt: If you frequently play music from Apple devices you will likely already use or be familiar with AirPlay. Let's take a look how to add AirPlay support to Snapcast.
---

 > If you haven't already be sure to read my [Introduction to Snapcast](https://scoot-technology.github.io/posts/2023-02-26-multi-room-audio-with-snapcast.html).

In previous posts we have looked at integrating different sources with Snapcast such as [turntables](https://scoot-technology.github.io/posts/2023-04-02-turntable-integration-with-snapcast.html) and [Chromecasts](https://scoot-technology.github.io/posts/2023-04-27-chromecast-integration-with-snapcast.html) and today we will cover [Apple AirPlay](https://www.apple.com/uk/airplay/).

If you own Apple devices you will likely already be familiar with AirPlay which enables you to stream what's playing on your phone over your network to an AirPlay endpoint. AirPlay 2 added the option of synchronising playback between multiple speakers but like with Google Cast this limits you to AirPlay capable speakers or devices. If we integrate an AirPlay endpoint with Snapcast we can stream music from our favourite apps and let Snapcast handle the rest.

# What Do I Need?
Unlike with Chromecast there are pure software implementations of AirPlay and AirPlay 2 for streaming audio which makes things much simpler. [shairport-sync](https://github.com/mikebrady/shairport-sync) is the most popular and the one I recommend for integration with Snapcast. As well as a solid AirPlay audio streaming implementation shairport-sync also has some useful features such as Apple Lossless support, an MQTT implementation which enables integration with the likes of Home Assistant and there is an official Docker image to get things up and running on Linux alongside Snapcast without too much trouble.

It's worth mentioning that Snapcast has [native support for shairport-sync](https://github.com/badaix/snapcast/blob/develop/doc/player_setup.md#airplay) which requires shairport-sync to be installed natively on your host via your distro's package manager or manually compiled from source. This works fine but there are a couple of drawbacks which put me off going this route.

 * The latest versions of shairport-sync with AirPlay 2 support don't work with Snapcast so you are limited to old releases.
 * The shairport-sync process has to be managed by Snapcast so you can't make configuration changes to one without restarting both.
 * Enabling more advanced features of shairport-sync like MQTT support is cumbersome or impossible using the native Snapcast method.
 * If you want to run everything with Docker then Snapcast and shairport-sync have to live in the same container so you can't use the official shairport-sync image.

So with that out of the way lets get into how I have things setup.

# AirPlay Setup
The following steps assume you will be running shairport-sync on the same host as your Snapcast server.

First up we will get our shairport-sync Docker container up and running using the [official image](https://hub.docker.com/r/mikebrady/shairport-sync). Take a look at the documentation in the aforementioned link which details several different options for starting a shairport-sync container.

For the most basic case run the following:

    docker run -d --net host -v /tmp:/tmp --name shairport-sync mikebrady/shairport-sync -a Snapcast -o pipe

For those who prefer **Docker Compose**:

    version: '3'
    services:
    shairport-sync:
        image: mikebrady/shairport-sync:latest
        network_mode: host
        restart: unless-stopped
        volumes:
        - /tmp:/tmp
        cap_add:
        - SYS_NICE
        command: -a Snapcast -o pipe

This will start a container running a shairport-sync instance which will appear on your network as an AirPlay device called **Snapcast** but you can change the name here as you like. We tell shairport-sync to use the **pipe** backend which will allow easy integration with Snapcast. If all is well shairport-sync will have created a FIFO pipe in `/tmp/shairport-sync-audio` which is where audio data will be sent.


# Snapcast Setup
Now we have our AirPlay client up and running we need to add it as a source in our Snapcast server.

Add the following to your Snapcast server configuration file:

    source = pipe:///tmp/shairport-sync-audio?name=AirPlay&mode=read

Restart your Snapcast server and you should see a new **AirPlay** source available.

<p align="center">
    <img src='{{ "assets/snapcast/snapcast_airplay_source.jpg" | relative_url }}' height="500">
</p>

At this point you can go ahead and start casting music from your favourite apps to your AirPlay instance and enjoy the music synchronized across all of your Snapcast clients.

<p align="center">
    <img src='{{ "assets/airplay/airplay_cast.jpg" | relative_url }}' height="500">
</p>