### Performance and Timing

_(Or, how long does `FastLED.show()` actually take, and will it break my other code?)_

[[Interrupt problems]] explains what goes wrong when `show()` collides with time-critical work. This page is the other half: **how to predict the numbers up front**, so you can evaluate a microcontroller for a project before you build it.

The good news is that clockless LED timing is not mysterious or measured — it is fixed by the chipset protocol and is computable to the microsecond. You do not need to profile it. You need one formula and one table.

---

## The formula

For any clockless (3-wire) chipset:

```
show() duration ≈ (bits_per_pixel × bit_period × num_pixels) + reset_time
```

- `bits_per_pixel` is **24** for RGB chipsets, **32** for RGBW (SK6812/WS2814/TM1814).
- `bit_period` is fixed by the chipset — every bit takes the same time, whether it is a 0 or a 1. That is how the protocol encodes data: the *ratio* of high-to-low varies, the *period* does not.
- `reset_time` is the low period that latches the frame. It is paid once per `show()`, not per pixel.

**Worked example — the classic case:**

100 WS2812 pixels: `24 × 1.25µs × 100 = 3000µs` of data, plus 280µs reset ≈ **3.3 ms**.

That is the number to plan around. It is not affected by brightness, by colour, by how much your effect code does, or by which platform you are on. It is the wire protocol.

---

## Per-pixel cost by chipset

All values below are **computed from FastLED's own timing table** (`src/fl/chipsets/led_timing.h`), which is the same data the drivers compile against. `bit period = T1 + T2 + T3`.

| Chipset | Bit period | Per pixel (24-bit) | Per pixel (32-bit RGBW) | Reset |
|---|---|---|---|---|
| **WS2812 / WS2812B** | 1250 ns | **30.00 µs** | — | 280 µs |
| WS2812B-V5 | 1225 ns | 29.40 µs | — | 280 µs |
| WS2813 | 1280 ns | 30.72 µs | — | 300 µs |
| WS2814 (RGBW) | 1280 ns | — | 40.96 µs | 300 µs |
| WS2815 | 1890 ns | 45.36 µs | — | 0 |
| **SK6812** | 1200 ns | 28.80 µs | 38.40 µs | 80 µs |
| SK6822 | 1750 ns | 42.00 µs | — | 0 |
| GS1903 | 1200 ns | 28.80 µs | — | 280 µs |
| SM16703 | 1200 ns | 28.80 µs | — | 0 |
| SM16824E | 1200 ns | 28.80 µs | — | 200 µs |
| TM1809 | 1150 ns | 27.60 µs | — | 0 |
| TM1814 (RGBW) | 1300 ns | — | 41.60 µs | 300 µs |
| TM1829 @ 800 kHz | 1230 ns | 29.52 µs | — | 500 µs |
| TM1829 @ 1600 kHz | 600 ns | 14.40 µs | — | 500 µs |
| PL9823 | 1710 ns | 41.04 µs | — | 0 |
| GE8822 | 1360 ns | 32.64 µs | — | 0 |
| LPD1886 @ 1250 kHz | 800 ns | 19.20 µs | — | 0 |
| UCS1903B @ 800 kHz | 1300 ns | 31.20 µs | — | 0 |
| UCS1904 @ 800 kHz | 1250 ns | 30.00 µs | — | 0 |
| UCS2903 | 1250 ns | 30.00 µs | — | 0 |
| UCS1912 | 1600 ns | 38.40 µs | — | 0 |
| UCS7604 @ 800 kHz | 1250 ns | 30.00 µs | — | 280 µs |
| UCS7604 @ 1600 kHz | 625 ns | 15.00 µs | — | 280 µs |
| **WS2811 @ 400 kHz** | 2500 ns | **60.00 µs** | — | 280 µs |
| UCS1903 @ 400 kHz | 2500 ns | 60.00 µs | — | 0 |
| TM1803 @ 400 kHz | 2500 ns | 60.00 µs | — | 0 |
| GW6205 @ 400 kHz | 2400 ns | 57.60 µs | — | 0 |
| GW6205 @ 800 kHz | 1200 ns | 28.80 µs | — | 0 |
| DP1903 @ 400 kHz | 3200 ns | 76.80 µs | — | 0 |
| DP1903 @ 800 kHz | 1800 ns | 43.20 µs | — | 0 |

Two things worth noticing:

- **400 kHz parts cost double.** If you have a choice between a WS2811 strip at 400 kHz and an 800 kHz part, that is a 2× difference in blackout time for the same pixel count.
- **RGBW costs 33% more than RGB** on the same chipset, because it is 32 bits per pixel instead of 24.

---

## Maximum frame rate

Frame rate is bounded by `1 / show() duration`, before your effect code costs anything:

| Pixels (WS2812) | `show()` | Ceiling |
|---|---|---|
| 8 | 0.52 ms | ~1900 fps |
| 50 | 1.78 ms | ~560 fps |
| 100 | 3.28 ms | ~305 fps |
| 300 | 9.28 ms | ~108 fps |
| 500 | 15.28 ms | ~65 fps |
| 1000 | 30.28 ms | ~33 fps |
| 2000 | 60.28 ms | ~17 fps |

At around **1000 pixels on a single data line** you are down to 33 fps and the protocol — not your microcontroller — is the bottleneck. No amount of CPU speed changes this. The fix is **parallel output**: driving N strips simultaneously divides the wall-clock time by N. See [[Parallel-Output]].

FastLED also enforces a per-chipset `getMaxRefreshRate()`, so calling `show()` faster than the chipset can latch will simply block until it is safe.

---

## The part that breaks other code: interrupt blackout

For clockless chipsets, the bit timing is enforced in software, so **interrupts must be held off** while data is going out — that is the whole of the [[Interrupt problems]] story. What varies is *how much* is held off, and that is a platform property.

FastLED's default per platform (from each `led_sysdefs_*.h`):

| Platform | `FASTLED_ALLOW_INTERRUPTS` default | What it means |
|---|---|---|
| **AVR** (Uno, Nano, Mega, Duemilanove…) | **0** | Interrupts off for the **entire** frame |
| **STM32** | **0** | Interrupts off for the entire frame |
| Teensy 3.x / 4.x, SAMD21/51, nRF51/52, RP2040/RP2350, SAM, Apollo3, Renesas, Giga, MGM240 | 1 | Re-enabled briefly **between pixels** |
| ESP8266 | 1 | Re-enabled between pixels, with frame retry |
| ESP32 | 1 | See below — usually not applicable |

**This is the single most important number for evaluating a microcontroller.**

On an **AVR**, a 100-pixel WS2812 frame means 3.3 ms with interrupts completely off. Any interrupt-driven peripheral — hardware serial, I²C, `Servo`, IR receive, `millis()` accuracy — is stalled for that entire window. This is exactly what @afaucher hit polling IR data on a Duemilanove.

On platforms with `FASTLED_ALLOW_INTERRUPTS 1`, interrupts get a window between each pixel. The constraint becomes: your ISR must complete in well under the per-pixel budget (a few µs), and must not need to fire more often than once per pixel period (~30 µs for WS2812). If it overruns, FastLED detects the corrupted frame and retries it — see `FASTLED_INTERRUPT_RETRY_COUNT`.

On **ESP32**, the default clockless path uses the RMT peripheral with DMA. The peripheral clocks the waveform out in hardware, so `show()` does not need to hold interrupts off for the frame at all. This is why an ESP32 can drive long strips alongside WiFi.

### Chipsets that avoid the problem entirely

**Clocked (4-wire) chipsets — APA102, SK9822, DotStar, LPD8806, WS2801 — have no timing requirement at all.** The clock line carries the timing, so an interrupt mid-frame is harmless; the transfer just pauses. FastLED never disables interrupts for these.

Their duration is set by SPI clock instead: `32 bits × num_pixels / SPI_clock`. At `DATA_RATE_MHZ(12)`, 100 APA102 pixels take ~270 µs — more than 10× faster than WS2812, with **zero** interrupt cost.

> If you have time-critical work and freedom to choose parts, this is the highest-leverage decision available to you.

---

## Evaluating a microcontroller for your project

A procedure you can run on paper before ordering anything:

1. **Compute your frame cost.** `bits × bit_period × pixels + reset`, from the table above.
2. **Find your tightest interrupt deadline.** For hardware serial at 57600 baud on AVR that is ~174 µs (one byte time, single-byte receive buffer). For a 38 kHz IR receiver it is tens of µs. For I²C it is set by the master's clock.
3. **Compare them.**
   - If frame cost < deadline: fine on any platform.
   - If frame cost > deadline **and** your platform defaults to `FASTLED_ALLOW_INTERRUPTS 0` (AVR, STM32): **you will lose data.** At 57600 baud on AVR that threshold is about 5 WS2812 pixels.
   - If frame cost > deadline but the platform allows interrupts: check your *ISR duration* against the per-pixel window instead of the whole frame.
4. **Check your frame-rate ceiling** against the table above.
5. **Check RAM.** 3 bytes per pixel for the `CRGB` array (4 for RGBW), plus your framebuffer if you keep one. On a 2 KB AVR that caps you near a few hundred pixels regardless of timing.

### If it does not fit

Roughly in order of effectiveness:

1. **Switch to a clocked chipset** (APA102/SK9822). Removes the interrupt problem entirely and is ~10× faster.
2. **Move to a platform with a hardware LED peripheral** — ESP32 (RMT/I2S/LCD_CAM/PARLIO), RP2040/RP2350 (PIO), Teensy 4 (FlexIO/ObjectFLED). These clock the waveform out in hardware.
3. **Use parallel output.** N strips at once divides wall-clock by N. See [[Parallel-Output]].
4. **Split into shorter strips** on separate controllers — shorter individual blackout windows.
5. **Call `show()` less often.** @afaucher's own fix: update on state change rather than every loop iteration. Often the cheapest change available.
6. **Prefer an 800 kHz part over a 400 kHz one** — an immediate 2× win.
7. **Move time-critical work off interrupts**, or onto a second core (ESP32, RP2040).

---

## Measuring it yourself

The formula predicts the wire time. If you want to measure the *whole* loop including your effect code, time it directly:

```cpp
void loop() {
    fl::u32 t0 = micros();
    myEffect();
    fl::u32 t1 = micros();
    FastLED.show();
    fl::u32 t2 = micros();
    Serial.print("effect: "); Serial.print(t1 - t0);
    Serial.print("us  show: "); Serial.println(t2 - t1);
}
```

Note that on AVR, `micros()` itself depends on a timer interrupt — so a `show()` that disables interrupts will make `micros()` *lose* time, not gain it. Measure across the whole loop, and treat the `show()` figure as a lower bound. On platforms that permit interrupts during `show()`, the reading is accurate.

`FastLED.getFPS()` reports the achieved frame rate directly, which is usually the number you actually care about.

---

## See also

- [[Interrupt problems]] — diagnosing serial/IR/servo data loss
- [[Parallel-Output]] — driving multiple strips simultaneously
- [[Platform limitations]] — per-platform constraints
- [[SPI Hardware or Bit banging]] — clocked chipset output paths
- [[Power notes]] — the *other* thing that limits large installations
