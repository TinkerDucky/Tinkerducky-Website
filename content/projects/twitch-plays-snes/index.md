---
title: "TwitchPlaysSNES"
date: 2024-04-23T23:44:42+02:00
lastmod: 2024-08-25T14:41:15+02:00
draft: false
---


## Real hardware instead of emulation

Playing video games via Twitch has become a thing at the latest since [TwitchPlaysPokemon](https://www.reddit.com/r/twitchplayspokemon/).
But while experiences like TwitchPlaysPokemon were realized with emulators on a PC, I wanted to create a gaming experience by remotely controlling a real SNES via Twitch chat commands.

After thinking about how to realise this project some challenges regarding the necessary software as well hardware occurred.

## Software Challenges
The controller software should has two main functions getting commands from the Twitch chat and representing a virtual controller which buttons states are affected by the chat messages to send them to the console frequently. 

Another challenge was due to the fact that I own a Japanese console with a Japanese version of the game (the US version of games like Chrono Trigger is extremely expensive). That results in just having Japanese texts making the understanding for many people very difficult. So, it should be possible to mark parts of the image which can be tried to translate.

All these aspects resulted to the follow requirements

1. Receive chat messages from the Twitch chat parsing them afterwards.
1. Creating a virtual representation of the button states within the software.
1. Sending the button states to the console according the [defined protocol](https://gamefaqs.gamespot.com/snes/916396-super-nintendo/faqs/5395) of the console.
1. Capturing the image of the console.
1. Defining a adjustable bounding box to mark an area within the captured image with text for translation.
1. Adding different image manipulation methods to improve visibiliy of text.
1. Applying [OCR](https://en.wikipedia.org/wiki/Optical_character_recognition) on the marked area.
1. Sending the recognized text to a translation service like [Deepl](https://www.deepl.com/de/translator)

### Receiving/parsing chat messages and virtual buttons

To tackle the challenge of receiving and parsing the chat messages I had to create two chat bots using the [Twitch API](https://dev.twitch.tv/docs/api/). Two chat bots are necessary due to the fact that one will has to be run on an embedded device connected with the SNES to receive the button pressed and forward them to the SNES (also called **SNES Bot**). The other one will also get the button pressed to show the current button states within the stream but will also be able to get additional commands to capture text from the current screen of the game for applying OCR and translating afterwards (also called **OCR/Translation Bot**).

Both bots will parse every message and check if some messages is a command to press a certain button. They will handle this on there own that means that there is a risk of inconsistency between there virtual buttons state. The worst what can happen is that the bot for ocr and translation will show a wrong button state (might happen if one loses the connection for a short time) but this does not affect the correct function of the whole system so adding another component for synchronizing makes too much time for now (maybe it will be added in the future)

#### SNES Bot

The current version of the Bot uses a ESP32 ([ESP-WROOM-32](Ehttps://www.espressif.com/sites/default/files/documentation/esp32-wroom-32e_esp32-wroom-32ue_datasheet_en.pdf)) combined with two [CD4021B shift registers](https://www.ti.com/product/CD4021B) to mimic the behaviour of a real controller. 

I also tried to emulate the behaviour in software because sending the buttons can be easily (in most causes) implemented because the protocol is well documented (see protocol link in requirement list) and not very complex. Every 16.67 ms (20ms for the PAL consoles) the SNES will send a short latch signal (12 μs long) to the controller containing a shift register (some versions also have two interconnected shift register) saving the current state of the pressed buttons. After a short time period (6 μs) the SNES will send 16 synchronic (50% duty cycle) clock pulses with a duration of 12 μs for each clock cycle. 

The only challenge is to react to the signals in time and that's where the problems occurred. The ESP32 has 7 different interrupt priorities and the typical interrupt have a priority between 1 and 3 ([ESP32 Interrupts](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/hlinterrupts.html)). But, interrupts with such a priority need too much time until the are invoked resulting in a wrong timing and therefore wrong inputs. It is possible to implement an interrupt routine with a higher priority but these have to be implemented in assembly and therefore I decided to implement it via hardware (maybe I will emulate it in software in the future)

### Capturing the image

Capturing the image seemed to be not a difficult task. There are a lot devices to capture the video and audio signals from different devices over various interfaces. After finding a proper I could get the the video data with help of [OpenCV](https://docs.opencv.org/3.4/d0/da7/videoio_overview.html) to use the captured frames for the further ocr and translation process.

