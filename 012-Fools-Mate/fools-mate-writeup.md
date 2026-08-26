# Fools Mate CTF Writeup

**TryHackMe:** [https://tryhackme.com/room/foolsmate](https://tryhackme.com/room/foolsmate)

## Overview

All the CTF provided was an address to a webpage: `http://MACHINE_IP`, With the task of finding 1 flag.

## 1. Reconnaissance

Heres the interface i was met with:

![[012-Fools-Mate/images/image0.png]]

The first thing i tried was just playing the game like normal, when i was about to checkmate the king i got this message:

![[012-Fools-Mate/images/image1.png]]

Since this is a 15 minute challenge on easy, i suspect all i have to do is inspect the `JavaScript` and remove the event that triggers the popup.

I spent some time looking through the `JavaScript` and I used the console to output some js variables but none of them contained the flag.

So i switched tactics and opened `BurpSuite` to see what i was actually sending the website when i made a move.

So i started the `BurpSuite Proxy` and when i tried to checkmate, nothing popped up. However when i move to a space that doesnt checkmate the king i send this request:

![[012-Fools-Mate/images/image2.png]]

So i tried just changing the position i moved to to be `a8` (the checkmate position), and then forwarded the request and when i went back on my browser, nothing happened. So i repeated those steps again but instead of forwarding it to the browser i sent the request to `repeater` so that i can see the exact html response from the webpage just to test it was even receiving my request correctly and i kept getting "error: illegal move" responses even on perfectly legal moves (not just checkmates). So i tried changing the cookie to see if it was something wrong with my session and it worked:

![[012-Fools-Mate/images/image3.png]]

## What i Learned
I learned that its easy to get tunnel visioned and sometimes when you've hit a few roadblocks the best thing you can do is take a step back and revise what other options you have. I also learned that doing seemingly random things like changing the session id cookie to a different value can be important when it comes to vulnerable and badly configured web applications.