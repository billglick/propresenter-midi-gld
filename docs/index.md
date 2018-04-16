
# Introduction
I’d heard of people using MIDI to help automate live production for a while, but hadn’t yet attempted to do it. I knew that our presentation software, ProPresenter, could be setup to send MIDI cues out to other devices (e.g. sound board, lighting system, click tracks, etc.). And since timing of lyrics and other presentations is very key to the flow of our church service, ProPresenter seemed like the obvious place to initiate MIDI automation.

But suddenly I had a scheduling emergency and I didn’t have a sound board operator for next Sunday. It was finally time to take a dive into the deep end of the pool and automate my Allen & Heath GLD-112 audio console via MIDI from ProPresenter. If I could get ProPresenter to cycle between a few preset scenes on the GLD, I could rest better with a less experienced audio engineer at the controls.

Well, it worked. And now I still use this same MIDI automation to control a few of our trickier audio cues from ProPresenter every week.

# Overview
...

## Our Environment and Equipment
Every week we use the following hardware and software:
* Allen & Heath GLD-112 sound console
* Apple iMac running latest MacOS 10.13 and ProPresenter 6.3

## Software Requirements for Mac
For the MIDI side of things, I had to add a few pieces of additional software:
* A&H TCP MIDI Driver for Mac (free)
* ProPresenter MIDI module
* Bome MIDI Translator Pro

## MIDI Over LAN Network
While you can setup an external USB MIDI interface on most computers, most modern MIDI works over LAN networking. Both my iMac and the GLD console support using MIDI over the LAN.

However, the trickiest part is that the GLD requires use of Allen & Heath’s TCP MIDI Driver for Mac in order to send MIDI to it over it’s LAN network interface. I couldn’t figure out a way to simply connect to the GLD as a networked MIDI device using Apple’s built-in audio tools.

## MIDI Channels
For future functionality, I decided to use different MIDI channels for various types of cues. For my purposes, I’m using MIDI channel #3 for sound cues. Below are the MIDI channels I’ve assigned for various types of cues.
1. Light Cues
2. Video Cues
3. Sound Cues

In the future, I plan to send lighting and video cues to other MIDI listening devices later on. But for sound, we’ll focus on cues on channel #3.

# GLD Setup
…

## LAN Network Setup
…

## MIDI Control Setup
...

# Mac Setup
...

## TCP MIDI Driver
After installing Allen & Heath’s TCP MIDI Driver for Mac, you have to start the A&H TCP MIDI Driver application, then click on the A&H icon in the status menus (upper right corner of screen) and select Preferences to set or find the IP address of your GLD console.

## MIDI Translation
The ProPresenter MIDI communication module supports only 2 types of MIDI commands: 1) “Note On”, and 2) “Note Off”. However, the Allen & Heath GLD MIDI API requires more than these two types of MIDI commands. For example, the GLD uses “Bank Change” and “Program Change” MIDI commands to recall scenes. So, there is no way to recall a scene on the GLD using only the ProPresenter MIDI communication module.

To get around this limitation we can use another piece of software that can listen for simpler MIDI commands from ProPresenter that then trigger more complex series of MIDI commands that can be sent to the GLD console. The software I chose is Bome MIDI Translator Pro.

Here’s an example of how this works:
1. ProPresenter sends a simple MIDI “Note On” command to the Bome MIDI Translator software installed and running on the same computer.
2. Bome MIDI Translator Pro then triggers a set of more complex MIDI commands that are then sent to the GLD console over the A&H MIDI TCP driver interface.
3. The GLD console listens over it’s LAN network interface for MIDI channel #3 and receives the series of MIDI commands from the Translator Pro software.

This lets me keep fairly simple cues for MIDI commands inside of ProPresenter, that then translate into a series of more complex MIDI commands in the MIDI Translator software.

## Programming Cues
I chose to map each action that I want to do on the GLD console to a different MIDI note and use the intensity of the note to determine what scene or channel to apply that action to. Currently I setup 3 types of actions:
1. (0) Scene Recall
2. (1) Mute Channel (including DCAs)
3. (2) Unmute channel (including DCAs

This allows me to recall scene #19 on the GLD console by sending a note of “0” with an intensity that maps to scene #19 to MIDI channel #3 (remember I’m using channel 3 for all sound cues) from ProPresenter. This command actually goes to the MIDI Translator software, which then listens for this command and responds by sending both a “Bank” and “Program” command to the GLD. 

| Action | ProPresenter Cue | Bome MIDI Translator  |
| ---    | ---              | ---                   |
| Scene Recall | Channel: 3<br>Note: C-2 (0)<br>Intensity: Scene # | Scene Select - Bank: 02, 00, 00<br>Scene Select - Program  02, oo - delay 10ms |
| Mute Channel | Channel: 3<br>Note: C#-2 (1)<br>Intensity: Channel # | Mute Channel - Note On: 02, oo, 7F<br>Mute Channel - Note Off:  02, oo, 00 - delay 10ms |
| Unmute Channel | Channel: 3<br>Note: D-2 (2)<br>Intensity: Channel # | Unmute Channel - Note On: 02, oo, 3F<br>Unmute Channel - Note Off:  02, oo, 00 - delay 10ms |

## Setup A&H MIDI TCP Driver
...

## Setup MIDI Translator Pro
…

## Setup ProPresenter MIDI Module
…

## Setup MIDI Cues in ProPresenter
…

# Usage Tips
Here are a few things that make this setup easier:
* During rehearsals it's a pain to have the GLD controlled by ProPresenter. The sound operator can quickly disable this by simply setting the GLD to listen on an an used MIDI channel. In our setup we generally flip over to MIDI channel #4 during rehearsal, then back to channel #3 for the service.
* Instead of manually entering MIDI cues into song presentations that change each week, I try to create general cues on slides that are consistent week-to-week. So, you might consider creating additional ‘song’ files that simply load the MIDI cues you want for each song, but then “GO TO NEXT” with a timing of zero.

# GLD MIDI Protocol
Allen & Heath provides GLD MIDI Protocol documentation to help you figure out what channels map to the MIDI channels.  But I find their documentation very hard to interpret, especially since they mostly refer to hexadecimal numbers instead of the decimal numbers used by MIDI. So, I tend to use my own tables for figuring out the primary input and scene numbers I make use of for cues.  Perhaps the following tables will help you figure out the input channels and scenes you care about.

| Channel            | Hexadecimal | Decimal |
| ---                | ---         | ---     |
| Input 17           | 30          | 48      |
| Stereo Input 45/46 | 4C          | 76      |
| Stereo Input 47/48 | 4E          | 78      |
| DCA 01             | 10          | 16      |
| DCA 02             | 11          | 17      |
| DCA 03             | 12          | 18      |
| DCA 04             | 13          | 19      |
| DCA 05             | 14          | 20      |
| DCA 06             | 15          | 21      |
| DCA 07             | 16          | 22      |
| DCA 08             | 17          | 23      |

| Scene | Hexadecimal | Decimal |
| ---   | ---         | ---     |
| 10    | 09          | 9       |
| 11    | 0A          | 10      |
| 12    | 0B          | 11      |
| 13    | 0C          | 12      |
| 14    | 0D          | 13      |
| 15    | 0E          | 14      |
| 16    | 0F          | 15      |
| 17    | 10          | 16      |
| 18    | 11          | 17      |

Or as a generic cheat method:
* Inputs = GLD input number + 31
* DCAs = GLD DCA number + 15
* Scenes = GLD scene number - 1


# Ideas for Improvements
Below are some general ideas that I (or you) may pursue to improve upon my initial implementation:
* Update the MIDI Translator actions to receive actual GLD channel or scene numbers, and translate the channel and scene numbers programatically in the MIDI Translator.
* Add additional action types for more GLD functionality.

