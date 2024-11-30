---
url: https://dev.to/miry/getting-started-with-obs-a-beginners-guide-129a
canonical_url: https://dev.to/miry/getting-started-with-obs-a-beginners-guide-129a
title: 'Getting Started with OBS: A Beginner''s Guide'
slug: getting-started-with-obs-a-beginners-guide-129a
description: Learn how to set up OBS on MacOS for screen recording and streaming.
  This article provides straightforward tips on microphone settings, screen capture,
  adding background music, and even achieving a professional look on video without
  a green screen. Start your streaming journey with OBS, even if you're not a pro
  – it's the ideal tool for beginners to create engaging content.
tags:
- obs
- streaming
- screencasting
author: Michael Nikitochkin
username: miry
---

![Cover image](/assets/2023-10-13-getting-started-with-obs-a-beginners-guide-129a-cover_image-43o6j687gvssudds800x.jpg)

# Getting Started with OBS: A Beginner's Guide


<small><i>Foto von <a href="https://unsplash.com/de/@maltehelmhold">Malte Helmhold</a> auf <a href="https://unsplash.com/de/fotos/AufvW7p8WM0">Unsplash</a></i></small>


## Introduction

If you're a beginner looking to dive into the world of live streaming and content creation, **OBS (Open Broadcaster Software)**[^1] is a powerful tool to help you get started. In this guide, I'll walk you through the basic steps to set up OBS on your MacBook Air M1 for recording screencasts and streaming content. While I'm not a professional streamer or content creator, I'll share some simple tips that can help you get up and running quickly. We'll skip some advanced topics like environment, lighting, and scripting, focusing on the essentials for beginners.

## Microphone Settings

One common challenge for beginners is how to record your voice while eliminating background noise. Not everyone has a soundproof studio, and recording might happen in less-than-ideal environments. I've faced similar issues with noise from servers, mechanical keyboards, mouse clicks, and even the sound of my own hands on the table.

To tackle this, I conducted some experiments with microphone settings. First, I adjusted the system settings for my microphone to capture primarily my voice and minimize other sounds. After some trial and error, I found that setting the sensitivity to around **70-80%** worked well.


![MacOS System Settings for Ton](/assets/2023-10-13-getting-started-with-obs-a-beginners-guide-129a-ksovzm2jyjvwytwz3j6z.png)

OBS provides additional audio settings with various devices, including **"Microphone/AUX-Audio"**. Although I found it didn't significantly alter the audio quality, disabling it did help slightly. Your mileage may vary.


![OBS Settings for Audio Mixer](/assets/2023-10-13-getting-started-with-obs-a-beginners-guide-129a-bwmu3smol4ez90yqnddj.png)

If you're still encountering background noise, OBS has a built-in noise suppression feature that can work wonders. Enabling **"Noise Suppression"** can effectively eliminate keyboard, mouse, and table sounds, providing cleaner audio.


![Audio Input capture Plugins Window](/assets/2023-10-13-getting-started-with-obs-a-beginners-guide-129a-kr3qcrrw4qxn4uok633p.png)

## Screen Capture

For most of my content, I'll want to record my screen. OBS excels at screen recording and even captures audio by default. However, one issue to watch out for is that OBS doesn't always pay attention to the global volume settings. For instance, it records the sound of applications at their native volume, which can sometimes be too loud.

To address this, you can adjust the audio settings for the **"Screen Capture"** source. I recommend setting it to -20 dB for applications with active audio and -40 dB if you want to include background music from streaming platforms.

## Background Music

Including background music in your content can enhance the viewing experience. There are two ways to do this: open a music application of your choice and adjust its volume, or add a "Media Source" in OBS. The latter allows you to loop a specific audio file or playlist.

You can source royalty-free music from websites like **Mixkit**[^2] and enhance the audio quality using plugins like an equalizer to fine-tune high, mid, and low frequencies.

## Video

If you want to appear on-camera in your videos but don't have access to a green screen, OBS offers a handy solution. The "Video" source allows you to add a live video feed. To remove your background, you can use plugins like the **Background Removal / Virtual Green-screen & Low-Light Enhance**[^3].

Keep in mind that this solution might not be as perfect as what you see on platforms like Google Meet. Achieving good results with the OBS plugin requires proper lighting, which helps separate you from the background.

![Background removal plugin settings page](/assets/2023-10-13-getting-started-with-obs-a-beginners-guide-129a-tnrilc3eu35s8lpj5vuk.png)

## Summary

OBS is a versatile tool that continues to improve and is ready to use from the moment you install it. With the tips and settings provided in this guide, you'll have an easy setup to start streaming on platforms like Twitch. While there's always room for improvement, OBS is a great choice for beginners looking to create engaging content.

## References

[^1]: [OBS Home page](https://obsproject.com/)
[^2]: [Mixkit](https://mixkit.co/free-stock-music/)
[^3]: [OBS Studio Plugin - Background Removal / Virtual Green-screen & Low-Light Enhance](https://obsproject.com/forum/resources/background-removal-virtual-green-screen-low-light-enhance.1260/)


