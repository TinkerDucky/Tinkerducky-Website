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

After thinking about how to realise this project some challenges regarding the necessary software as well hardware occurred.

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

#### Receiving/parsing chat messages and virtual buttons

To tackle the challenge of receiving and parsing the chat messages we have to create two chat bots using the [Twitch API](https://dev.twitch.tv/docs/api/). Two chat bots are necessary due to the fact that one will has to be run on an embedded device connected with the SNES to receive the button pressed and forward them to the SNES (also called **SNES Bot**). The other one will also get the button pressed to show the current button states within the stream but will also be able to get additional commands to capture text from the current screen of the game for applying OCR and translating afterwards (also called **OCR/Translation Bot**).

Both bots will parse every message and check if some messages is a command to press a certain button. They will handle this on there own that means that there is a risk of inconsistency between there virtual buttons state. The worst what can happen is that the bot for ocr and translation will show a wrong button state (might happen if one loses the connection for a short time) but this does not affect the correct function of the whole system so adding another component for synchronizing makes too much time for now (maybe it will be added in the future)

#### Sending buttons states to the SNES

Sending the buttons is in most cases easy because the protocol is well documented (see protocol link in requirement list) and not very complex. Every 16.67 ms (20ms for the PAL consoles) the SNES will send a short latch signal (12 μs long) to the controller containing a shift register (some versions also have two interconnected shift register) saving the current state of the pressed buttons. After a short time period (6 μs) the SNES will send 16 synchronic (50% duty cycle) clock pulses with a duration of 12 μs for each clock cycle.

With this information and todays microcontrollers it seems to be no problem to simulate this behavior within the software. So, we just need a microcontroller which is capable of connecting to the internet to communicate with Twitch and send the current button states to the SNES if it is asking for. In theory this approach will work like a charm but the reality shows that programming embedded software brings many obstacles with it which will be explained more later.

#### Capturing the image

Capturing the image will also does not sound like a difficult task. There are a lot devices to capture the video and audio signals from different devices over various interfaces. After finding a proper one you can get the video with help of [OpenCV](https://docs.opencv.org/3.4/d0/da7/videoio_overview.html) use the captured frames for the further ocr and translation process.

