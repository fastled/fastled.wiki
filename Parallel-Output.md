**N.B. - this code is currently only available on the FastLED3.1 branch and may set your leds on fire!**
# Parallel Output on WS2811 leds

WS2812 strips are slow for writing data, with a data rate of just 800khz, it takes 30µs to write out a single led's worth of data.  One way to improve the performance here is with parallel output, driving 8 lines in parallel gets you, effectively, 8 times the data rate.  One of the first libraries to do this for WS2811 leds (if not the first) is the [OctoWS2811 library for the Teensy 3/3.1](http://www.pjrc.com/teensy/td_libs_OctoWS2811.html).  This library is especially optimized for taking data from USB and shoving it out to the leds, effectively using the Teensy 3.x as a dumb frame buffer.

OctoWS2811 has two limitations for the average FastLED user, however.  The first is that OctoWS2811 uses its own functions for setting color data, and doesn't provide any of the brightness, color correction, math functions, etc... that FastLED does.  In addition, while you could mix libraries and use FastLED to prep your color data then call into OctoWS2811's setpixel functions, for a 64x8 array of leds this would end up taking nearly 2ms worth of calls to setPixel - which is nearly the amount of time it takes to write 512 leds worth of WS2811 data, removing much of the benefit of OctoWS2811.  The second limitation to OctoWS2811 is that it is limited to 8 lines, and a specific set of pins, and the Teensy 3.

## Multi-platform Parallel output

If you are using a due or a digix or a teensy 3 or a teensy 3.1, FastLED now has some new parallel output controllers that will allow you to drive 8 lines of WS2812 strips in parallel.  This means that instead of taking 15.3ms/frame of CPU time to write out 512 bytes of data, it would take closer to 1.9ms/frame. See examples/Multiple/ParallelOutputDemo to see how this works.  

On the teensy 3 and 3.1 there's two sets of pins that we can use for parallel output, described below:

* WS2811_PORTD - the OctoWS2811 pins - 2,14,7,8,6,20,21,5
* WS2811_PORTC - pins 15,22,23,9,10,13,11,12,28,27,29,30 (yes, 12 pins!  If you're willing to solder onto pads on the back of the teensy)
* WS2811_PORTDC - pins 2,14,7,8,6,20,21,5,15,22,23,9,10,13,11,12 <-- 16 pins, no soldering onto pads on the back!

On the due/digix (note: some pins aren't available on the due, which means anything you write to that block of led data will just disappear):

* WS2811_PORTA - pins 69, 68, 61,60,59, 100, 58, 31 (pin 100 only available on the digix)
* WS2811_PORTC - pins 90, 91, 92, 93, 94, 95, 96, 97 (only available on the digix)
* WS2811_PORTD - pins 25,26,27,28,14,15,29,11 (all available on the due)

And on the ESP8266:

* WS2811_PORTA - pins 12, 13, 14 and 15 (or pins 6,7,5 and 8 on the NodeMCU boards/pin layout).

You can also replace WS2811 with WS2811_400, TM1803, and UCS1903 in the above.

## Making OctoWS2811 faster with FastLED

If you only have 8 lines of leds, and you are using a Teensy 3/3.1, there's a lot of benefit to using OctoWS2811.  Wanting to encourage people to get the most benefit of what is available to them, FastLED 3.1 introduces an OctoWS2811 controller.  This code preps OctoWS2811's drawing buffer far faster than using setPixel, and gives you the benefit of FastLED's scaling/dithering/color correction code.  How much faster?  For a 64x8 led setup, 500µs vs. 2ms.  You can see this in action in examples/Multiple/OctoWS2811Demo

A comparison of CPU timings, for writing out 512 leds:

* 512 leds on 8 pins, w/FastLED3.0: 15,360µs or 66fps max
* 512 leds on 8 pins, w/OctoWS2811 using setPixel: 2000µs or 500fps max
* 512 leds on 8 pins, w/FastLED3.1 parallel output: 1900µs or 526fps max
* 512 leds on 8 pins, w/FastLED3.1 feeding OctoWS2811: 500µs or 2000fps max (sadly, WS2811's get unhappy if you go above 400-500fps)

