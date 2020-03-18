---
layout: post
title: CyberThreat 2019 Badge Writeup
---

Last year I was lucky enough to attend the inaugural CyberThreat conference put on by NCSC and SANS and it was also the first time I was introduced to interactive badges at a conference.

![CyberThreat 2018 Badge](https://www.1234n6.com/content/images/2020/03/IMG_20191128_220503.jpg)

The badges formed the bases of a CTF, which was one of the highlights of the event as far as I was concerned. Despite a concerted effort during the two day conference, nobody was able to complete all the associated challenges before the event ended. I was pleased to find out when I completed it 2 hours after the event had ended that nobody had beaten me to the chase and I could claim the prize.

For those interested, a video walkthrough from James Lyne ([@jameslyne](https://twitter.com/jameslyne)) and Simon McNameee ([@mcnamee_simon](https://twitter.com/mcnamee_simon)), that details the various stages, is available below:

Unsurprisingly having secured a place at CyberThreat 2019 and having seen various comments suggesting that the badges were going to be bigger and better than last year, I was excited to have a play.

This year, rather than being a mere two hours late, it was a full two days after the conference had closed before I was able to complete the final challenge. I don't think I will have been the first, but nevertheless it was good fun.

What follows is a brief write up of the process followed, with much of the error excluded and much of the luck written to make it sound like I have more of a clue than I do.
## The Badges
This year the badges had clearly had something of an upgrade:

![CyberThreat 2019 Badge](https://www.1234n6.com/content/images/2020/03/IMG_20191125_094740.jpg)

The single button input and 4 LED output were now upgraded in a GameBoyesque fashion. An LCD with configurable display Name/Alias, timeout and backlight colour, as well as SDCard and multiple input buttons, made for a considerably more versatile interface for challenges.

Within the device you had a basic menu which offered 'Settings' or 'Challenges', the latter of which was the CTF with 5 unlockable challenges:

![Challenge List](https://www.1234n6.com/content/images/2020/03/IMG_20191125_094543.jpg)
## Getting Started
It's probably worth pointing out that before starting the CTF, I immediately dropped the SD card out of the badge and imaged it, because I'm a forensicator and that's what forensicators do right!? But, as I had suspected it might, this proved to be handy later.

Further to this, throughout the CTF I had the badge connected via USB and was monitoring it over serial using my computer. This was based on my previous experience from last year, where all interaction with the badge occurred in this way.

Connecting to the badge can be achieved in a number of ways but the first requirement is to identify the correct COM port.

### In Windows:
Within device manager, after connecting the device via USB we can review what ports are in use:

![Device Manager](https://www.1234n6.com/content/images/2020/03/dev.png)

In this case COM5.

### In Linux:

There are a number of ways to confirm this but I generally grep dmesg for 'tty' after connecting the device.

`dmesg | grep tty`

Once we know the COM port in use we can use Putty, Arduino IDE, screen, python serial or a myriad of other methods to communicate with the badge. I found Arduino IDE with 9600 baud to be reliable for interactive needs, screen via WSL was also helpful and for later challenges pySerial was needed.
## Level 1 - Maze Madness
The first level 'Maze Madness' presents you with a current location, goal location, a score and the simple instruction "Press 'A' to move!".

![Maze Madness - Just before completion](https://www.1234n6.com/content/images/2020/03/s.jpg)

Being the suspicious type, I promptly tried every button other than A, and quickly learned that B exited the game and no other buttons did anything noticable. I also tried the 'Konami Code' but was not rewarded with the instant win I had hoped for...

Next up I pressed the 'A' button and found that my location changed. Spamming it a few times seemed to move the coordinates seemingly randomly with any 1 press resulting in either X+1, X-1, Y+1 or Y-1. When pressing the button repeatedly it became apparent that the direction of movement was rotating North, East, South, West and that quick repeated presses and pauses could be used to move the Location in the rough direction desired.

Following some patience and luck I was eventually rewarded with my first win!

![A winner is me](https://www.1234n6.com/content/images/2020/03/IMG_20191125_125659.jpg)
## Level 2 - Close Proximity
Level 2 presented the player with a scrollable wall of hex, which in the case of my badge was 3 screens worth (192 bytes).

![Wall of Hex](https://www.1234n6.com/content/images/2020/03/IMG_20191128_225405.jpg)

Notably we also see communication over the serial interface we are monitoring with a simple request to enter a password.

![screen](https://www.1234n6.com/content/images/2020/03/screen.png)

After I had recovered from the 'SEC503 flashback' that being presented with a surprise wall of hex tends to induce, I began the process of transcribing the values so I could work on determining what they represented. Incidentally, if you are a CTF author and you are considering having participants transcribe 384 characters... don't.

![Typing this out was ace](https://www.1234n6.com/content/images/2020/03/3be4652f-648d-4046-acb2-a07b1edfcfc5.jpg)

Once we look at the ASCII representation of the hex a few things jump out.

Firstly we can see "Adam H harrisonamj" as well as some other sub-strings of the same (presumably resulting from multiple changes to the names I configured on my badge).

We also see references to 'BKRGB', 'NAMES', 'CHALL', 'KEY' and 'FLAG'... and we all like flags. What we have between offset 80h and B0h is a "page directory esque structure"...
