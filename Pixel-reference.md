# Overview

There's two main pixel types in the library - the CRGB class and the CHSV class.  CHSV objects have to be converted to CRGB objects before they can be written out.  You can also write CHSV objects into the CRGB array and have the translation occur as necessary.  

* [CRGB Reference](#crgb)
* [CHSV Object Reference](#chsv)
* [Predefined Colors Reference](#colors)

<h1 id="crgb">CRGB Reference</h1>

<wiki:toc max_depth="3" />

A "CRGB" is an object representing a color in RGB color space.  It contains simply:
 * a one byte value (0-255) representing the amount of red,
 * a one byte value (0-255) representing the amount of green,
 * a one byte value (0-255) representing the amount of blue
in a given color.  

Typically, when using this library, each LED strip is represented as an array of CRGB colors, one color for each LED pixel.
```
  #define NUM_LEDS 160

  CRGB leds[ NUM_LEDS ];
```

For more general information on what the RGB color space is, see http://en.wikipedia.org/wiki/RGB_color_model

# Data Members

CRGB has three one-byte data members, each representing one of the three red, green, and blue color channels of the color.  There is more than one way to access the RGB data; each of these following examples does exactly the same thing:
```
  // The three color channel values can be referred to as "red", "green", and "blue"...
  leds[i].red   = 50;
  leds[i].green = 100;
  leds[i].blue  = 150;

  // ...or, using the shorter synonyms "r", "g", and "b"...
  leds[i].r = 50;
  leds[i].g = 100;
  leds[i].b = 150;

  // ...or as members of a three-element array:
  leds[i][0] = 50;  // red
  leds[i][1] = 100; // green
  leds[i][2] = 150; // blue
```

## Direct Access
You are welcome, and invited, to directly access the underlying memory of this object if that suits your needs.  That is to say, there is no "CRGB::setRed( myRedValue )" method; instead you just directly store 'myRedValue' into the ".red" data member on the object. All of the methods on the CRGB class expect this, and will continue to operate normally.  This is a bit unusual for a C++ class, but in a microcontroller environment this can be critical to maintain performance.

The CRGB object "is trivially copyable", meaning that it can be copied from one place in memory to another and still function normally.

# Methods

In addition to simply providing data storage for the RGB colors of each LED pixel, the CRGB class also provides several useful methods color-manipulation, some of which are implemented in assembly language for speed and compactness.  Often using the class methods described here is faster and smaller than hand-written C/C++ code to achieve the same thing.

## Setting RGB Colors

CRGB colors can be set by assigning values to the individual red, green, and blue channels.  In addition, CRGB colors can be set a number of other ways which are often more convenient and compact.  The two pieces of code below perform the exact same function.
```
  //Example 1: set color from red, green, and blue components individually
  leds[i].red =    50;
  leds[i].green = 100;
  leds[i].blue =  150;

  //Example 2: set color from red, green, and blue components all at once
  leds[i] = CRGB( 50, 100, 150);
```
Some performance-minded programmers may be concerned that using the 'high level', 'object-oriented' code in the second example comes with a penalty in speed or code size.  However, this is simply not the case;  the examples above generate literally identical machine code, taking up exactly the same amount of program memory, and executing in exactly the same amount of time.  Given that, the choice of which way to write the code, then, is entirely a matter of personal taste and style.  All other things being equal, the simpler, higher-level, more object-oriented code is generally recommended.

Here are the other high-level ways to set a CRGB color in one step:
```
  // Example 3: set color via 'hex color code' (0xRRGBB)
  leds[i] = 0xFF007F;

  // Example 4: set color via any named HTML web color
  leds[i] = CRGB::HotPink;

  // Example 5: set color via setRGB
  leds[i].setRGB( 50, 100, 150);
```
Again, for the performance-minded programmer, it's worth noting that all of the examples above compile down into exactly the same number of machine instructions.  Choose the method that makes your code the simplest, most clear, and easiest to read and modify.

Colors can also be copied from one CRGB to another:
```
  // Copy the CRGB color from one pixel to another
  leds[i] = leds[j];
```
If you are copying a large number of colors from one (part of an) array to another, the standard library function memmove can be used to perform a bulk transfer; the CRGB object "is trivially copyable".
```
  // Copy ten led colors from leds[src .. src+9] to leds[dest .. dest+9]
  memmove( &leds[dest], &leds[src], 10 * sizeof( CRGB) );
```
Performance-minded programmers using AVR/ATmega MCUs to move large number of colors in this way may wish to use the alternative "memmove8" library function, as it is measurably faster than the standard libc "memmove".

<h2 id="chsv">Setting HSV Colors </h2>

### Introduction to HSV
CRGB color objects use separate red, green, and blue channels internally to represent each composite color, as this is exactly the same way that multicolor LEDs do it: they have one red LED, one green LED, and one blue LED in each 'pixel'.  By mixing different amounts of red, green, and blue, thousands or millions of resultant colors can be displayed.

However, working with raw RGB values in your code can be awkward in some cases.  For example, it is difficult to work express different tints and shades of a single color using just RGB values, and it can be particular daunting to describe a 'color wash' in RGB that cycles around a rainbow of hues while keeping a constant brightness.

To simplify working with color in these ways, the library provides access to an alternate color model based on three different axes: Hue, Saturation, and Value (or 'Brightness').  For a complete discussion of HSV color, see http://en.wikipedia.org/wiki/HSL_and_HSV , but briefly:
 * Hue is the 'angle' around a color wheel
 * Saturation is how 'rich' (versus pale) the color is
 * Value is how 'bright' (versus dim) the color is

In the library, the "hue" angle is represented as a one-byte value ranging from 0-255.  It runs from red to orange, to yellow, to green, to aqua, to blue, to purple, to pink, and back to red. Here are the eight cardinal points of the hue cycle in the library, and their corresponding hue angle.

![FastLED Rainbow color map](https://raw.github.com/FastLED/FastLED/gh-pages/images/HSV-rainbow-with-desc.jpg)
[Click here for full-size chart.](https://raw.github.com/FastLED/FastLED/gh-pages/images/HSV-rainbow-with-desc.jpg)

 * Red (0..) "HUE_RED"
 * Orange (32..) "HUE_ORANGE"
 * Yellow (64..) "HUE_YELLOW"
 * Green (96..) "HUE_GREEN"
 * Aqua (128..) "HUE_AQUA"
 * Blue (160..) "HUE_BLUE"
 * Purple (192..) "HUE_PURPLE"
 * Pink(224..) "HUE_PINK"

Often in other HSV color spaces, hue is represented as an angle from 0-360 degrees.  But for compactness, efficiency, and speed, this library represents hue as a single-byte number from 0-255.  There's a full [wiki page how FastLED deals with HSV colors here](FastLED-HSV-Colors).

"saturation" is a one-byte value ranging from 0-255, where 255 means "completely saturated, pure color", 128 means "half-saturated, a light, pale color", and 0 means "completely de-saturated: plain white".

"value" is a one-byte value ranging from 0-255 representing brightness, where 255 means "completely bright, fully lit", 128 means "somewhat dimmed, only half-lit", and zero means "completely dark: black."

### The CHSV Object

In the library, a CHSV object is used to represent a color in HSV color space.  The CHSV object has the three one-byte data members that you might expect:
 * hue (or 'h')
 * saturation (or 'sat', or just 's')
 * value (or 'val', or just 'v')
These can be directly manipulated in the same way that red, green, and blue can be on a CRGB object.  CHSV objects are also "trivially copyable".

```
  // Set up a CHSV color
  CHSV paleBlue( 160, 128, 255);

  // Now...
  //   paleBlue.hue == 160
  //   paleBlue.sat == 128
  //   paleBlue.val == 255
```

### Automatic Color Conversion
The library provides fast, efficient methods for converting a CHSV color into a CRGB color. Many of these are automatic and require no explicit code.

For example, to set an led to a color specified in HSV, you can simply assign a CHSV color to a CRGB color:
```
  // Set color from Hue, Saturation, and Value.  
  // Conversion to RGB is automatic.
  leds[i] = CHSV( 160, 255, 255);

  // alternate syntax
  leds[i].setHSV( 160, 255, 255);

  // set color to a pure, bright, fully saturated, hue
  leds[i].setHue( 160);
```

There is no conversion back from CRGB to CHSV provided with the library at this point.

### Explicit Color Conversion

There are two different HSV color spaces: "spectrum" and "rainbow", and they're not exactly the same thing.  Wikipedia has a good discussion here http://en.wikipedia.org/wiki/Rainbow#Number_of_colours_in_spectrum_or_rainbow but for purposes of the library, it can be summed up as follows:
 * "Spectra" have barely any real yellow in them; the yellow band is incredibly narrow.
 * "Rainbows" have a band of yellow approximately as wide as the 'orange' and 'green' bands around it; the yellow range is easy to see.

All of the automatic color conversions in the library use the "HSV Rainbow" color space, but through use of explicit color conversion routines, you can select to use the "HSV Spectrum" color space.  There's a full [wiki page how FastLED deals with HSV colors here](FastLED-HSV-Colors).

The first explicit color conversion function is hsv2rgb_rainbow, which is used in the automatic color conversions:
```
  // HSV (Rainbow) to RGB color conversion
  CHSV hsv( 160, 255, 255); // pure blue in HSV Rainbow space
  CRGB rgb;
  hsv2rgb_rainbow( hsv, rgb);
  // rgb will now be (0, 0, 255)  -- pure blue as RGB
```

The HSV Spectrum color space has different cardinal points, and only six of them, which are correspondingly spread out further numerically.  Here is the "Spectrum" color map that FastLED provides if you call hsv2rgb_spectrum explicitly:
![FastLED Spectrum color map](https://raw.github.com/FastLED/FastLED/gh-pages/images/HSV-spectrum-with-desc.jpg)
[Click here for full-size chart.](https://raw.github.com/FastLED/FastLED/gh-pages/images/HSV-spectrum-with-desc.jpg)

 * Red (0..)
 * Yellow (42..)
 * Green (85..)
 * Aqua (128..)
 * Blue (171..)
 * Purple (213..)

The hsv2rgb_spectrum conversion function's API is identical to hsv2rgb_rainbow:
```
  // HSV (Spectrum) to RGB color conversion
  CHSV hsv( 171, 255, 255); // pure blue in HSV Spectrum space
  CRGB rgb;
  hsv2rgb_spectrum( hsv, rgb);
  // rgb will now be (0, 0, 255)  -- pure blue as RGB
```

Why use the Spectrum color space, instead of Rainbow?  The HSV Spectrum color space can be converted to RGB a little faster than the HSV Rainbow color space can be -- but the results are not as good visually; what little yellow there is appears dim, and at lower brightnesses, almost brownish.  So there is a trade-off between a few clock cycles and visual quality.  In general, start with the Rainbow functions (or better yet, the automatic conversions), and drop down to the Spectrum functions only if you completely run out of speed.

Both color space conversion functions can also convert an array of CHSV colors to a corresponding array of CRGB colors:
```
  // Convert ten CHSV rainbow values to ten CRGB values;
  CHSV hsvs[10];
  CRGB leds[10];
  // (set hsv values here)
  hsv2rgb_rainbow( hsvs, leds, 10); // convert all
```
The function "hsv2rgb_spectrum" can also be called this way for bulk conversions.

## Comparing Colors

CRGB colors can be compared for exact matches using == and !=.

CRGB colors can be compared for relative light levels using <, >, <=, and =>.  Note that this is a simple numeric comparison, and it will not always match the perceived brightness of the colors.  

Often it is useful to check if a color is completely 'black', or if it is 'lit' at all.  You can do this by testing the color directly with 'if', or using it in any other boolean context.
```
  // Test if a color is lit at all (versus pure black)
  if( leds[i] ) {
    /* it is somewhat lit (not pure black) */
  } else {
    /* it is completely black */
  }
```

## Color Math

The library supports a rich set of 'color math' operations that you can perform on one or more colors.  For example, if you wanted to add a little bit of red to an existing LED color, you could do this:
```
  // Here's all that's needed to add "a little red" to an existing LED color:
  leds[i] += CRGB( 20, 0, 0);
```

That's it.  

If you've ever done this sort of thing by hand before, you may notice something missing: the check for the red channel overflowing past 255.  Traditionally, you've probably had to do something like this:
```
  // Add a little red, the old way.
  uint16_t newRed;
  newRed = leds[i].r + 20;
  if( newRed > 255) newRed = 255; // prevent wrap-around
  leds[i].r = newRed;
```
This kind of add-and-then-check-and-then-adjust-if-needed logic is taken care of for you inside the library code for adding two CRGB colors, inside operator+ and operator+=.  Furthermore, much of this logic is implemented directly in assembly language and is substantially smaller and faster than the corresponding C/C++ code.  The net result is that you no longer have to do all the checking yourself, and your program runs faster, too.  

These 'color math' operations are part of what makes the library fast: it lets you develop your code faster, as well as executing it faster.

All of the math operations defined on the CRGB colors are automatically protected from wrap-around, overflow, and underflow.

### Adding and Subtracting Colors
```
  // Add one CRGB color to another.
  leds[i] += CRGB( 20, 0, 0);

  // Add a constant amount of brightness to all three (RGB) channels.
  leds[i].addToRGB(20);

  // Add a constant "1" to the brightness of all three (RGB) channels.
  leds[i]++;


  // Subtract one color from another.
  leds[i] -= CRGB( 20, 0, 0);

  // Subtract a contsant amount of brightness from all three (RGB) channels.
  leds[i].subtractFromRGB(20);

  // Subtract a constant "1" from the brightness of all three (RGB) channels.
  leds[i]--;
```
### Dimming and Brightening Colors

There are two different methods for dimming a color: "video" style and "raw math" style.  Video style is the default, and is explicitly designed to never accidentally dim any of the RGB channels down from a lit LED (no matter how dim) to an UNlit LED -- because that often comes out looking wrong at low brightness levels.  The "raw math" style will eventually fade to black.

Colors are always dimmed down by a fraction.  The dimming fraction is expressed in 256ths, so if you wanted to dim a color down by 25% of its current brightness, you first have to express that in 256ths.  In this case, 25% = 64/256.

```
  // Dim a color by 25% (64/256ths)
  // using "video" scaling, meaning: never fading to full black
  leds[i].fadeLightBy( 64 );
```

You can also express this the other way: that you want to dim the pixel to 75% of its current brightness.   75% = 192/256.  There are two ways to write this, both of which will do the same thing.  The first uses the %= operator; the rationale here is that you're setting the new color to "a percentage" of its previous value:
```
  // Reduce color to 75% (192/256ths) of its previous value
  // using "video" scaling, meaning: never fading to full black
  leds[i] %= 192;
```
The other way is to call the underlying scaling function directly.  Note the "video" suffix.
```
  // Reduce color to 75% (192/256ths) of its previous value
  // using "video" scaling, meaning: never fading to full black
  leds[i].nscale8_video( 192);
```

If you want the color to eventually fade all the way to black, use one of these functions:
```
  // Dim a color by 25% (64/256ths)
  // eventually fading to full black
  leds[i].fadeToBlackBy( 64 );

  // Reduce color to 75% (192/256ths) of its previous value
  // eventually fading to full black
  leds[i].nscale8( 192);
```

A function is also provided to boost a given color to maximum brightness while keeping the same hue:
```
  // Adjust brightness to maximum possible while keeping the same hue.
  leds[i].maximizeBrightness();
```

Finally, colors can also be scaled up or down using multiplication and division.
```
  // Divide each channel by a single value
  leds[i] /= 2;

  // Multiply each channel by a single value
  leds[i] *= 2;
```

### Constraining Colors Within Limits
The library provides a function that lets you 'clamp' each of the RGB channels to be within given minimums and maximums.  You can force all of the color channels to be at least a given value, or at most a given value.  These can then be combined to limit both minimum and maximum.
```
  // Bring each channel up to at least a minimum value.  If any channel's
  // value is lower than the given minimum for that channel, it is
  // raised to the given minimum.  The minimum can be specified separately
  // for each channel (as a CRGB), or as a single value.
  leds[i] |= CRGB( 32, 48, 64);
  leds[i] |= 96;


  // Clamp each channel down to a maximum value.  If any channel's
  // value is higher than the given maximum for that channel, it is
  // reduced to the given maximum.  The minimum can be specified separately
  // for each channel (as a CRGB), or as a single value.
  leds[i] &= CRGB( 192, 128, 192);
  leds[i] &= 160;

```
### Misc Color Functions
The library provides a function that 'inverts' each RGB channel.  Performing this operation twice results in the same color you started with.
```
  // Invert each channel
  leds[i] = -leds[i];
```
The library also provides functions for looking up the apparent (or mathematical) brightness of a color.
```
  // Get brightness, or luma (brightness, adjusted for eye's sensitivity to
  // different light colors.   See http://en.wikipedia.org/wiki/Luma_(video) )
  uint8_t luma = leds[i].getLuma();
  uint8_t avgLight = leds[i].getAverageLight();
```

### Predefined colors list

***Please note*** - these predefined colors are defined using the W3C RGB definitions.  These definitions are designed with RGB monitors in mind, not RGB leds, and so the colors that you get on LED strips may be a bit different than you're expecting.  In our experience, the colors are often too pale, or washed out (overly-desaturated).


| Color name|Hex Value|Example
|-----------|---------|-------
| CRGB::AliceBlue | 0xF0F8FF | <img src="http://www.colorcombos.com/images/colors/hex/F0F8FF.png"/>
| CRGB::Amethyst | 0x9966CC | <img src="http://www.colorcombos.com/images/colors/hex/9966CC.png"/>
| CRGB::AntiqueWhite | 0xFAEBD7 | <img src="http://www.colorcombos.com/images/colors/hex/FAEBD7.png"/>
| CRGB::Aqua | 0x00FFFF | <img src="http://www.colorcombos.com/images/colors/hex/00FFFF.png"/>
| CRGB::Aquamarine | 0x7FFFD4 | <img src="http://www.colorcombos.com/images/colors/hex/7FFFD4.png"/>
| CRGB::Azure | 0xF0FFFF | <img src="http://www.colorcombos.com/images/colors/hex/F0FFFF.png"/>
| CRGB::Beige | 0xF5F5DC | <img src="http://www.colorcombos.com/images/colors/hex/F5F5DC.png"/>
| CRGB::Bisque | 0xFFE4C4 | <img src="http://www.colorcombos.com/images/colors/hex/FFE4C4.png"/>
| CRGB::Black | 0x000000 | <img src="http://www.colorcombos.com/images/colors/hex/000000.png"/>
| CRGB::BlanchedAlmond | 0xFFEBCD | <img src="http://www.colorcombos.com/images/colors/hex/FFEBCD.png"/>
| CRGB::Blue | 0x0000FF | <img src="http://www.colorcombos.com/images/colors/hex/0000FF.png"/>
| CRGB::BlueViolet | 0x8A2BE2 | <img src="http://www.colorcombos.com/images/colors/hex/8A2BE2.png"/>
| CRGB::Brown | 0xA52A2A | <img src="http://www.colorcombos.com/images/colors/hex/A52A2A.png"/>
| CRGB::BurlyWood | 0xDEB887 | <img src="http://www.colorcombos.com/images/colors/hex/DEB887.png"/>
| CRGB::CadetBlue | 0x5F9EA0 | <img src="http://www.colorcombos.com/images/colors/hex/5F9EA0.png"/>
| CRGB::Chartreuse | 0x7FFF00 | <img src="http://www.colorcombos.com/images/colors/hex/7FFF00.png"/>
| CRGB::Chocolate | 0xD2691E | <img src="http://www.colorcombos.com/images/colors/hex/D2691E.png"/>
| CRGB::Coral | 0xFF7F50 | <img src="http://www.colorcombos.com/images/colors/hex/FF7F50.png"/>
| CRGB::CornflowerBlue | 0x6495ED | <img src="http://www.colorcombos.com/images/colors/hex/6495ED.png"/>
| CRGB::Cornsilk | 0xFFF8DC | <img src="http://www.colorcombos.com/images/colors/hex/FFF8DC.png"/>
| CRGB::Crimson | 0xDC143C | <img src="http://www.colorcombos.com/images/colors/hex/DC143C.png"/>
| CRGB::Cyan | 0x00FFFF | <img src="http://www.colorcombos.com/images/colors/hex/00FFFF.png"/>
| CRGB::DarkBlue | 0x00008B | <img src="http://www.colorcombos.com/images/colors/hex/00008B.png"/>
| CRGB::DarkCyan | 0x008B8B | <img src="http://www.colorcombos.com/images/colors/hex/008B8B.png"/>
| CRGB::DarkGoldenrod | 0xB8860B | <img src="http://www.colorcombos.com/images/colors/hex/B8860B.png"/>
| CRGB::DarkGray | 0xA9A9A9 | <img src="http://www.colorcombos.com/images/colors/hex/A9A9A9.png"/>
| CRGB::DarkGreen | 0x006400 | <img src="http://www.colorcombos.com/images/colors/hex/006400.png"/>
| CRGB::DarkKhaki | 0xBDB76B | <img src="http://www.colorcombos.com/images/colors/hex/BDB76B.png"/>
| CRGB::DarkMagenta | 0x8B008B | <img src="http://www.colorcombos.com/images/colors/hex/8B008B.png"/>
| CRGB::DarkOliveGreen | 0x556B2F | <img src="http://www.colorcombos.com/images/colors/hex/556B2F.png"/>
| CRGB::DarkOrange | 0xFF8C00 | <img src="http://www.colorcombos.com/images/colors/hex/FF8C00.png"/>
| CRGB::DarkOrchid | 0x9932CC | <img src="http://www.colorcombos.com/images/colors/hex/9932CC.png"/>
| CRGB::DarkRed | 0x8B0000 | <img src="http://www.colorcombos.com/images/colors/hex/8B0000.png"/>
| CRGB::DarkSalmon | 0xE9967A | <img src="http://www.colorcombos.com/images/colors/hex/E9967A.png"/>
| CRGB::DarkSeaGreen | 0x8FBC8F | <img src="http://www.colorcombos.com/images/colors/hex/8FBC8F.png"/>
| CRGB::DarkSlateBlue | 0x483D8B | <img src="http://www.colorcombos.com/images/colors/hex/483D8B.png"/>
| CRGB::DarkSlateGray | 0x2F4F4F | <img src="http://www.colorcombos.com/images/colors/hex/2F4F4F.png"/>
| CRGB::DarkTurquoise | 0x00CED1 | <img src="http://www.colorcombos.com/images/colors/hex/00CED1.png"/>
| CRGB::DarkViolet | 0x9400D3 | <img src="http://www.colorcombos.com/images/colors/hex/9400D3.png"/>
| CRGB::DeepPink | 0xFF1493 | <img src="http://www.colorcombos.com/images/colors/hex/FF1493.png"/>
| CRGB::DeepSkyBlue | 0x00BFFF | <img src="http://www.colorcombos.com/images/colors/hex/00BFFF.png"/>
| CRGB::DimGray | 0x696969 | <img src="http://www.colorcombos.com/images/colors/hex/696969.png"/>
| CRGB::DodgerBlue | 0x1E90FF | <img src="http://www.colorcombos.com/images/colors/hex/1E90FF.png"/>
| CRGB::FireBrick | 0xB22222 | <img src="http://www.colorcombos.com/images/colors/hex/B22222.png"/>
| CRGB::FloralWhite | 0xFFFAF0 | <img src="http://www.colorcombos.com/images/colors/hex/FFFAF0.png"/>
| CRGB::ForestGreen | 0x228B22 | <img src="http://www.colorcombos.com/images/colors/hex/228B22.png"/>
| CRGB::Fuchsia | 0xFF00FF | <img src="http://www.colorcombos.com/images/colors/hex/FF00FF.png"/>
| CRGB::Gainsboro | 0xDCDCDC | <img src="http://www.colorcombos.com/images/colors/hex/DCDCDC.png"/>
| CRGB::GhostWhite | 0xF8F8FF | <img src="http://www.colorcombos.com/images/colors/hex/F8F8FF.png"/>
| CRGB::Gold | 0xFFD700 | <img src="http://www.colorcombos.com/images/colors/hex/FFD700.png"/>
| CRGB::Goldenrod | 0xDAA520 | <img src="http://www.colorcombos.com/images/colors/hex/DAA520.png"/>
| CRGB::Gray | 0x808080 | <img src="http://www.colorcombos.com/images/colors/hex/808080.png"/>
| CRGB::Green | 0x008000 | <img src="http://www.colorcombos.com/images/colors/hex/008000.png"/>
| CRGB::GreenYellow | 0xADFF2F | <img src="http://www.colorcombos.com/images/colors/hex/ADFF2F.png"/>
| CRGB::Honeydew | 0xF0FFF0 | <img src="http://www.colorcombos.com/images/colors/hex/F0FFF0.png"/>
| CRGB::HotPink | 0xFF69B4 | <img src="http://www.colorcombos.com/images/colors/hex/FF69B4.png"/>
| CRGB::IndianRed | 0xCD5C5C | <img src="http://www.colorcombos.com/images/colors/hex/CD5C5C.png"/>
| CRGB::Indigo | 0x4B0082 | <img src="http://www.colorcombos.com/images/colors/hex/4B0082.png"/>
| CRGB::Ivory | 0xFFFFF0 | <img src="http://www.colorcombos.com/images/colors/hex/FFFFF0.png"/>
| CRGB::Khaki | 0xF0E68C | <img src="http://www.colorcombos.com/images/colors/hex/F0E68C.png"/>
| CRGB::Lavender | 0xE6E6FA | <img src="http://www.colorcombos.com/images/colors/hex/E6E6FA.png"/>
| CRGB::LavenderBlush | 0xFFF0F5 | <img src="http://www.colorcombos.com/images/colors/hex/FFF0F5.png"/>
| CRGB::LawnGreen | 0x7CFC00 | <img src="http://www.colorcombos.com/images/colors/hex/7CFC00.png"/>
| CRGB::LemonChiffon | 0xFFFACD | <img src="http://www.colorcombos.com/images/colors/hex/FFFACD.png"/>
| CRGB::LightBlue | 0xADD8E6 | <img src="http://www.colorcombos.com/images/colors/hex/ADD8E6.png"/>
| CRGB::LightCoral | 0xF08080 | <img src="http://www.colorcombos.com/images/colors/hex/F08080.png"/>
| CRGB::LightCyan | 0xE0FFFF | <img src="http://www.colorcombos.com/images/colors/hex/E0FFFF.png"/>
| CRGB::LightGoldenrodYellow | 0xFAFAD2 | <img src="http://www.colorcombos.com/images/colors/hex/FAFAD2.png"/>
| CRGB::LightGreen | 0x90EE90 | <img src="http://www.colorcombos.com/images/colors/hex/90EE90.png"/>
| CRGB::LightGrey | 0xD3D3D3 | <img src="http://www.colorcombos.com/images/colors/hex/D3D3D3.png"/>
| CRGB::LightPink | 0xFFB6C1 | <img src="http://www.colorcombos.com/images/colors/hex/FFB6C1.png"/>
| CRGB::LightSalmon | 0xFFA07A | <img src="http://www.colorcombos.com/images/colors/hex/FFA07A.png"/>
| CRGB::LightSeaGreen | 0x20B2AA | <img src="http://www.colorcombos.com/images/colors/hex/20B2AA.png"/>
| CRGB::LightSkyBlue | 0x87CEFA | <img src="http://www.colorcombos.com/images/colors/hex/87CEFA.png"/>
| CRGB::LightSlateGray | 0x778899 | <img src="http://www.colorcombos.com/images/colors/hex/778899.png"/>
| CRGB::LightSteelBlue | 0xB0C4DE | <img src="http://www.colorcombos.com/images/colors/hex/B0C4DE.png"/>
| CRGB::LightYellow | 0xFFFFE0 | <img src="http://www.colorcombos.com/images/colors/hex/FFFFE0.png"/>
| CRGB::Lime | 0x00FF00 | <img src="http://www.colorcombos.com/images/colors/hex/00FF00.png"/>
| CRGB::LimeGreen | 0x32CD32 | <img src="http://www.colorcombos.com/images/colors/hex/32CD32.png"/>
| CRGB::Linen | 0xFAF0E6 | <img src="http://www.colorcombos.com/images/colors/hex/FAF0E6.png"/>
| CRGB::Magenta | 0xFF00FF | <img src="http://www.colorcombos.com/images/colors/hex/FF00FF.png"/>
| CRGB::Maroon | 0x800000 | <img src="http://www.colorcombos.com/images/colors/hex/800000.png"/>
| CRGB::MediumAquamarine | 0x66CDAA | <img src="http://www.colorcombos.com/images/colors/hex/66CDAA.png"/>
| CRGB::MediumBlue | 0x0000CD | <img src="http://www.colorcombos.com/images/colors/hex/0000CD.png"/>
| CRGB::MediumOrchid | 0xBA55D3 | <img src="http://www.colorcombos.com/images/colors/hex/BA55D3.png"/>
| CRGB::MediumPurple | 0x9370DB | <img src="http://www.colorcombos.com/images/colors/hex/9370DB.png"/>
| CRGB::MediumSeaGreen | 0x3CB371 | <img src="http://www.colorcombos.com/images/colors/hex/3CB371.png"/>
| CRGB::MediumSlateBlue | 0x7B68EE | <img src="http://www.colorcombos.com/images/colors/hex/7B68EE.png"/>
| CRGB::MediumSpringGreen | 0x00FA9A | <img src="http://www.colorcombos.com/images/colors/hex/00FA9A.png"/>
| CRGB::MediumTurquoise | 0x48D1CC | <img src="http://www.colorcombos.com/images/colors/hex/48D1CC.png"/>
| CRGB::MediumVioletRed | 0xC71585 | <img src="http://www.colorcombos.com/images/colors/hex/C71585.png"/>
| CRGB::MidnightBlue | 0x191970 | <img src="http://www.colorcombos.com/images/colors/hex/191970.png"/>
| CRGB::MintCream | 0xF5FFFA | <img src="http://www.colorcombos.com/images/colors/hex/F5FFFA.png"/>
| CRGB::MistyRose | 0xFFE4E1 | <img src="http://www.colorcombos.com/images/colors/hex/FFE4E1.png"/>
| CRGB::Moccasin | 0xFFE4B5 | <img src="http://www.colorcombos.com/images/colors/hex/FFE4B5.png"/>
| CRGB::NavajoWhite | 0xFFDEAD | <img src="http://www.colorcombos.com/images/colors/hex/FFDEAD.png"/>
| CRGB::Navy | 0x000080 | <img src="http://www.colorcombos.com/images/colors/hex/000080.png"/>
| CRGB::OldLace | 0xFDF5E6 | <img src="http://www.colorcombos.com/images/colors/hex/FDF5E6.png"/>
| CRGB::Olive | 0x808000 | <img src="http://www.colorcombos.com/images/colors/hex/808000.png"/>
| CRGB::OliveDrab | 0x6B8E23 | <img src="http://www.colorcombos.com/images/colors/hex/6B8E23.png"/>
| CRGB::Orange | 0xFFA500 | <img src="http://www.colorcombos.com/images/colors/hex/FFA500.png"/>
| CRGB::OrangeRed | 0xFF4500 | <img src="http://www.colorcombos.com/images/colors/hex/FF4500.png"/>
| CRGB::Orchid | 0xDA70D6 | <img src="http://www.colorcombos.com/images/colors/hex/DA70D6.png"/>
| CRGB::PaleGoldenrod | 0xEEE8AA | <img src="http://www.colorcombos.com/images/colors/hex/EEE8AA.png"/>
| CRGB::PaleGreen | 0x98FB98 | <img src="http://www.colorcombos.com/images/colors/hex/98FB98.png"/>
| CRGB::PaleTurquoise | 0xAFEEEE | <img src="http://www.colorcombos.com/images/colors/hex/AFEEEE.png"/>
| CRGB::PaleVioletRed | 0xDB7093 | <img src="http://www.colorcombos.com/images/colors/hex/DB7093.png"/>
| CRGB::PapayaWhip | 0xFFEFD5 | <img src="http://www.colorcombos.com/images/colors/hex/FFEFD5.png"/>
| CRGB::PeachPuff | 0xFFDAB9 | <img src="http://www.colorcombos.com/images/colors/hex/FFDAB9.png"/>
| CRGB::Peru | 0xCD853F | <img src="http://www.colorcombos.com/images/colors/hex/CD853F.png"/>
| CRGB::Pink | 0xFFC0CB | <img src="http://www.colorcombos.com/images/colors/hex/FFC0CB.png"/>
| CRGB::Plaid | 0xCC5533 | <img src="http://www.colorcombos.com/images/colors/hex/CC5533.png"/>
| CRGB::Plum | 0xDDA0DD | <img src="http://www.colorcombos.com/images/colors/hex/DDA0DD.png"/>
| CRGB::PowderBlue | 0xB0E0E6 | <img src="http://www.colorcombos.com/images/colors/hex/B0E0E6.png"/>
| CRGB::Purple | 0x800080 | <img src="http://www.colorcombos.com/images/colors/hex/800080.png"/>
| CRGB::Red | 0xFF0000 | <img src="http://www.colorcombos.com/images/colors/hex/FF0000.png"/>
| CRGB::RosyBrown | 0xBC8F8F | <img src="http://www.colorcombos.com/images/colors/hex/BC8F8F.png"/>
| CRGB::RoyalBlue | 0x4169E1 | <img src="http://www.colorcombos.com/images/colors/hex/4169E1.png"/>
| CRGB::SaddleBrown | 0x8B4513 | <img src="http://www.colorcombos.com/images/colors/hex/8B4513.png"/>
| CRGB::Salmon | 0xFA8072 | <img src="http://www.colorcombos.com/images/colors/hex/FA8072.png"/>
| CRGB::SandyBrown | 0xF4A460 | <img src="http://www.colorcombos.com/images/colors/hex/F4A460.png"/>
| CRGB::SeaGreen | 0x2E8B57 | <img src="http://www.colorcombos.com/images/colors/hex/2E8B57.png"/>
| CRGB::Seashell | 0xFFF5EE | <img src="http://www.colorcombos.com/images/colors/hex/FFF5EE.png"/>
| CRGB::Sienna | 0xA0522D | <img src="http://www.colorcombos.com/images/colors/hex/A0522D.png"/>
| CRGB::Silver | 0xC0C0C0 | <img src="http://www.colorcombos.com/images/colors/hex/C0C0C0.png"/>
| CRGB::SkyBlue | 0x87CEEB | <img src="http://www.colorcombos.com/images/colors/hex/87CEEB.png"/>
| CRGB::SlateBlue | 0x6A5ACD | <img src="http://www.colorcombos.com/images/colors/hex/6A5ACD.png"/>
| CRGB::SlateGray | 0x708090 | <img src="http://www.colorcombos.com/images/colors/hex/708090.png"/>
| CRGB::Snow | 0xFFFAFA | <img src="http://www.colorcombos.com/images/colors/hex/FFFAFA.png"/>
| CRGB::SpringGreen | 0x00FF7F | <img src="http://www.colorcombos.com/images/colors/hex/00FF7F.png"/>
| CRGB::SteelBlue | 0x4682B4 | <img src="http://www.colorcombos.com/images/colors/hex/4682B4.png"/>
| CRGB::Tan | 0xD2B48C | <img src="http://www.colorcombos.com/images/colors/hex/D2B48C.png"/>
| CRGB::Teal | 0x008080 | <img src="http://www.colorcombos.com/images/colors/hex/008080.png"/>
| CRGB::Thistle | 0xD8BFD8 | <img src="http://www.colorcombos.com/images/colors/hex/D8BFD8.png"/>
| CRGB::Tomato | 0xFF6347 | <img src="http://www.colorcombos.com/images/colors/hex/FF6347.png"/>
| CRGB::Turquoise | 0x40E0D0 | <img src="http://www.colorcombos.com/images/colors/hex/40E0D0.png"/>
| CRGB::Violet | 0xEE82EE | <img src="http://www.colorcombos.com/images/colors/hex/EE82EE.png"/>
| CRGB::Wheat | 0xF5DEB3 | <img src="http://www.colorcombos.com/images/colors/hex/F5DEB3.png"/>
| CRGB::White | 0xFFFFFF | <img src="http://www.colorcombos.com/images/colors/hex/FFFFFF.png"/>
| CRGB::WhiteSmoke | 0xF5F5F5 | <img src="http://www.colorcombos.com/images/colors/hex/F5F5F5.png"/>
| CRGB::Yellow | 0xFFFF00 | <img src="http://www.colorcombos.com/images/colors/hex/FFFF00.png"/>
| CRGB::YellowGreen | 0x9ACD32 | <img src="http://www.colorcombos.com/images/colors/hex/9ACD32.png"/>
