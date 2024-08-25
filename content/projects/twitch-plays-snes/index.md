---
title: "TwitchPlaysSNES"
date: 2024-04-23T23:44:42+02:00
lastmod: 2024-08-25T14:41:15+02:00
draft: false
---


## Real hardware instead of emulation

Playing video games via Twitch has become a thing at the latest since [TwitchPlaysPokemon](https://www.reddit.com/r/twitchplayspokemon/).
But while experiences like TwitchPlaysPokemon were realized with emulators on a PC, I wanted to create a gaming experience by remotely controlling a real SNES via Twitch chat commands.

## The Challenges

After thinking about how to realise this project some challenges regarding the necessary software as well hardware occured.

### Software Challenges
The controller software shall has two main functions getting commands from the Twitch chat and representing a virtual controller which buttons states are affected by the chat messages to send them to the console frequently. 

Another challenge is due to the fact that I own a Japanese console with a Japanese version of Chrono Trigger (the US version is extremely expensive). That results in just having Japanese texts making the understanding for many people very difficult. So, it shall be possible to mark parts of the image which can be tried to translate.

All these aspects results to the follow requirements

 
1. Receive chat messages from the Twitch chat parsing them afterwards.
1. Creating a virtual representation of the button states within the software.
1. Sending the button states to the console according the [defined protocol](https://gamefaqs.gamespot.com/snes/916396-super-nintendo/faqs/5395) of the console.
1. Capturing the image of the console.
1. Defining a adjustable bounding box to mark an area within the captured image with text for translation.
1. Adding different image manipulation methods to improve visibiliy of text.
1. Applying [OCR](https://en.wikipedia.org/wiki/Optical_character_recognition) on the marked area.
1. Sending the recognized text to a translation service like [Deepl](https://www.deepl.com/de/translator)