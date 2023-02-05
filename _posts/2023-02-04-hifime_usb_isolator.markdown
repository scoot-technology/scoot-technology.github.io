---
layout: post
title:  "Hifime USB Isolator"
date:   2023-02-04
categories: [Hi-Fi, Computing]
feature_image: assets/hifime_usb_isolator/hifime_usb_isolator_side.jpg
excerpt: Do USB isolation devices really work? I tested the cheapest and most basic one from Hifime to find out...
---

<p align="center">
    <img src='{{ "assets/hifime_usb_isolator/hifime_usb_isolator_top.jpg" | relative_url }}' >
</p>

I've seen USB isolation devices for sale in various places but generally dismissed them given USB interference isn't something I can recall being a problem. After all any good DAC or audio interface surely has USB interference factored into the design of the product? In over 15 years working with USB based audio devices I have never had a noticeable issue with interference caused by ground loops... but then I hooked up my Audio Technica AT-LP5x turntable using the USB output!

# What's The Problem?

It's worth mentioning that even without USB connected the AT-LP5x turntable in my system isn't completely silent using the built-in preamp. There is a noticeable hum when the preamp is powered but it's not excessive. However, when I hooked up the USB output to my computer the hum became considerably worse to the point of being intolerable. To make matters worse if I also connected a HDMI cable to the computer the hum was joined by a high frequency squeal. I tested a few different computers and the interference ranged from virtually nothing to horrendous so this definitely won't be an issue for everyone but it's also clear that the turntable preamp has no isolation against USB interference. Given the sensitivity of a preamp and the gains involved in it's operation any interference however small can easily become a significant problem...

# What's The Solution?

I decided to do some research into USB isolation and found several devices from the likes of [AudioQuest](https://www.audioquest.com/), [iFi Audio](https://ifi-audio.com/) and [Hifime](https://hifimediy.com/) all of which vary wildly in terms of price and claimed functionality. Some simply target ground loops and work by creating an independent ground for the USB device so it doesn’t have to share ground with your computer. Generally if your intention is just to reduce interference and improve the noise floor of your USB device this is likely all you need however more comprehensive devices also filter additional interference like RF and improve jitter by re-clocking the signal.

Not wanting to spend crazy money I took a punt on a basic [USB isolator](https://hifimediy.com/product-category/usb-isolators/) from [Hifime](https://hifimediy.com/) to see if this solved the issue. Given the turntable in this instance doesn't require power from USB and also isn't a high-speed device I went for the cheapest variant which only supports USB low and full speed modes and can supply up to 200mA of power. There is a [handy guide](https://hifimediy.com/2020/06/choosing-the-right-usb-isolator/) on the Hifime website to help understand which isolator will be required for your needs.

# What's The Verdict?

Getting things setup is a very simple process and doesn't require any drivers or special wizardry regardless of what computer and operating system you are running. The USB isolator simply passes through the data lines of the USB interface only focussing on power and ground so in theory your computer or connected device shouldn't know it is in the chain at all. The USB isolator accepts a standard USB-A input and comes pre-fitted with a USB-A connector at the other end. I connected my turntable to the USB isolator with a standard USB-B to USB-A cable and then plugged the isolator into my computers USB port as normal. As if by magic there was no noticeable increase in background noise or interference as there had been previously. I tried several times with and without the USB isolator connected and it clearly made a huge difference to the interference I was encountering lowering the noise floor to the same discernable levels as having USB disconnected altogether from the turntable. I have had my turntable connected through the USB isolator for over a week and haven't noticed any disconnections or data issues in that time despite extensive use so all in all it's a big thumbs up from me.


# Conclusion

Being a relatively cheap and simple device I wondered if the [Hifime USB Isolator](https://hifimediy.com/product/usb-isolator/) would be sufficient to isolate all of the interference I was encountering but I'm pleased to say for me it worked brilliantly. I have no doubt your mileage may vary depending on what kind of interference you are encountering. Things like HDMI and WiFi or Bluetooth radios, especially on cheap single-board computers, can be especially problematic so something with enhanced filtering may be required. That said, I would also like to think that more expensive preamps with USB outputs for digitizing records might also have better isolation than what is generally found on more entry-level lifestyle turntables but I digress...

If like me you are suffering with annoying audible interference with a USB audio device then I recommend giving a [Hifime USB isolator](https://hifimediy.com/product-category/usb-isolators/) a try!