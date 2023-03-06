---
layout: post
title:  "Snapcast Automatic Source Selection"
date:   2023-03-01
categories: [Computing, Snapcast]
feature_image: assets/snapcast/snapcast_feature_image.jpg
excerpt: Learn how meta sources can be used to make Snapcast source selection automatic.
---

If you are thinking of moving from Google Cast, AirPlay or similar to [Snapcast](https://github.com/badaix/snapcast) there are a few things about the user experience that can be frustrating or at least more cumbersome, especially if you have lots of audio sources configured. For example if you were to use Google Cast from a supported app on your phone to play music on your Chromecast supported speaker the experience is generally pretty seamless and you can control everything from one interface. If you then decide to cast something from a different app it's pretty straight forward.

With Snapcast whenever I want to switch between sources I have to go to my Snapcast control interface and manually switch the source of each client which can be cumbersome. Luckily Snapcast has the ability to detect when a source is idle or playing and coupled with [meta sources](https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#meta) you can have your Snapcast clients automatically switch to the active source without any manual interaction.

 > If you haven't already be sure to read my [introduction to Snapcast](https://scoot-technology.github.io/posts/2023-02-26-multi-room-audio-with-snapcast.html) blog post

# Meta Sources

Lets say you have the following sources setup in your Snapcast Server:

 * Music
 * Chromecast
 * AirPlay

If I am playing some music from my local collection with MPD and then begin casting something from an app on my phone I would also have to go to the Snapcast app, web interface or Home Assistant to change the source of each client from Music to Chromecast. I can configure a [meta source](https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#meta) to automate this step and make the experience more seamless.

Here is an example of a [meta source](https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#meta) in the Snapcast Server configuration file:

    source = meta:///Chromecast/AirPlay/Music?name=Auto

This will create an additional source called **Auto** which combines existing sources in priority order. In this instance **Chromecast** has the highest priority so if multiple sources are active whatever is playing on the **Chromecast** source will be heard if the **Auto** source is selected for a given client.

<p align="center">
    <img src='{{ "assets/snapcast/snapcast_meta_source.jpg" | relative_url }}' >
</p>

One note of caution is that if your audio source is always in a **Playing** state for some reason automatic switching won't work unless this source is lowest in the priority order. For capture devices configured using an [Alsa source](https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#alsa) with Snapcast you need to ensure **idle_threshold** is set to detect silence and change the source state accordingly. If using some other audio source over TCP or pipe which will continue to stream audio even when silent it's important to include a filter to remove silence and effectively stop sending audio unless something is actually playing.

# Conclusion

So there you have it, [meta sources](https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#meta), a simple solution to what can be a cumbersome experience when using [Snapcast](https://github.com/badaix/snapcast).

By configuring a [meta source](https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#meta) for my most frequently used audio sources I can have my clients automatically play what I want without any manual interaction. I can switch from playing music from my local collection to streaming something over AirPlay and my Snapcast clients seamlessly oblige.