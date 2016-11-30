#### 0. Is there documentation?  Where can I get help?

FastLED is a large, complex library.  It is closer to a framework than a simple library like most Arduino libraries you may be used to using so far.  Documentation and explanation of what the library can do, and how to do complex things with it is an ongoing project/process for us.  However, there are lots of pieces in place:

* [This wiki](http://fastled.io/wiki) - read through the pages on the sidebar here, there's a lot of information in there!
* [API Documentation](http://fastled.io/docs/3.1/index.html) - doxygen generated API documentation for the library.  This is more reference than how-to, and is continually being updated/expanded
* [The examples](https://github.com/FastLED/FastLED/tree/master/examples) - FastLED has many examples to show off how to do things in the library.  These are also continually expanding.
* [The G+ Group](http://fastled.io/+) - FastLED has a large community of active users, many of whom are quite helpful.  The community is open to people of all levels, and we've helped with everything from basics of programming to hardware design, as well as using FastLED.  Google recently broke searching in communities, but if you want to search the FastLED community, [start here](https://plus.google.com/s/FastLED/top), and add what you want to search for to the search bar at top.
* [The FastLED chatroom](http://fastled.io/chat) - the chat room is a place where some random conversation about FastLED occurs.  

#### 0a. Guidelines for asking for help on the G+ group

* Be complete!  Describe what is going on.  "It doesn't work" isn't helpful.  "The first 10 leds light up, but the remaining 30 don't" is better.
* Did we mention details?  Please also include:
  * The version of the arduino IDE you are using (and what OS you're using)
  * The version of FastLED you are using
  * The LED chipset you are using
  * What hardware you are building for
* Providing a link to your code will get some of the quickest help, as we don't have to guess at how you're trying to do what you're doing.  Please upload your code [gist](http://gist.github.com) so we can read it (code in G+ posts gets unreadable very quickly).  Also please update your entire sketch.  You will often be wrong about where you think the problem is in your code.  Please do _not_ use pastebin anymore, as this appears to cause posts to get auto-moderated by g+... thanks, google.
* [More on asking good questions](http://stackoverflow.com/help/how-to-ask) and [making examples](http://stackoverflow.com/help/mcve)

#### 1. I'm losing serial data when I call FastLED.show(), why?

Short version - you're running into problems with interrupts.  Long version - see the [[Interrupt problems]] wiki page.

#### 2. Help!  When I say leds[0] = CRGB::Red it comes out green!

Not all LED chipsets receive their data in RGB order.  See [[RGB Calibration]] for more info on how to adjust the rgb ordering.

#### 3. Help!  I'm getting a compiler error about "avr/io.h" not being found!

This most likely means that you are compiling for a platform that FastLED doesn't yet support.  Please [check the FastLED issues](http://fastled.io/issues) to see if there's already an issue for your platform, and if not, [make a new issue to add your platform](https://github.com/FastLED/FastLED/issues/new).  

#### 4. Help!  I'm getting weird flickering with my leds, and nothing looks right, why?

First thing to check is the wiring.  Make sure that you have power going to the + (or vcc) line on your leds.  Make sure that you have ground from your power going to the - (or gnd) line on your leds.  Also, make sure that you are running ground to ground on your controller as well, especially if you are running your leds off of a separate power supply than your controller.  Next, check your data lines:

Are your data (or data and clock) lines going to DIN (or DIN and CIN) on the leds?  If you connect the data line from your arduino to the DOUT (data out) pin on your leds, nothing is going to come out right.

Likewise, if you are using a 4-wire chipset like the APA102, if you have connected your clock line to your data line and vice versa, then Weird Stuff™ is going to happen.

#### 4b. I've corrected my wiring but now nothing lights up!

For some LED chipsets, if you accidentally wire ground and power backwards, or if you apply power to DIN, you will blow out the led.  _*You have not destroyed your entire strip of leds*_.  Luckily, you've most likely only damaged the first led in the chain, and you can cut it out and re-connect your wiring to the second led and continue using the rest of the leds.

#### 5. With APA102 leds, my wiring is right, but my leds are flickering.  (Or my leds start flickering somewhere down the line).

APA102 leds allow for high data rates.  I've driven them at 24Mhz+ for nearly 1000 leds.  However, for some reason, some ways of manufacturing APA102 strips have problems with high data rates when the strip is long.  If this happens, you can try slowing down the data rate that FastLED uses to write out APA102 data.  Often, setting it to 12Mhz or 10Mhz works:

```
FastLED.addLeds<APA102,7,11,RGB,DATA_RATE_MHZ(10)>(leds,NUM_LEDS);
```

#### 6. Why doesn't FastLED support this random led chipset or that random MCU?

There's a couple possible reasons why FastLED may not support a particular LED chipset or run on a particular MCU:

* Time - this is actually the most likely reason.  FastLED's primary developers have full time jobs, as well as fairly full plates of led work (personal art, contract development, ongoing FastLED development).  This means the list of things we want to do is growing faster than our ability to work on things.
* Awareness - sometimes we just aren't aware of a new LED chipset or MCU yet.  Feel free to [open an issue](https://github.com/FastLED/FastLED/issues/new) to put something on our radar.
* Technical - some LED chipsets we have made a decision to not directly support in the library.  Primarily this includes any LED chipset that requires interrupts/timers to properly manage/control (HL1606, LPD6803).  FastLED supports multiple AVR variants, as well as nearly a half dozen arm architectures (with a couple architectures that are neither ARM or AVR on the horizon as well) - each of which do interrupts and timers slightly differently.  Likewise, with some MCU architectures we've made a decision to not support because of either time (msp430, for example) or because we don't feel the platform is a useful platform to try to optimize FastLED for (PIC).
