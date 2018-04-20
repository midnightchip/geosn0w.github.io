<b>Abstract</b>

There are times when you need to take a closer look at the address space of your Arduino development board. 
Small sketches may or may not render memory problems depending on the Arduino board you've got, but a fairly complex project can 
easily chew through the SRAM available and multiple allocations and deallocations using `malloc()` and `free()` may result in 
memory issues because the available free chunks may not be big enough to hold what you're about to allocate.

In situations like these, it's recommended to keep an eye on the memory, the way data is positioned, and how you can optimize it.

### A matter of memory

As you probably know, Arduino boards, especially the `AVR` ones, don't come with an awful lot of memory. They're intended for relatively small projects (or very well optimized ones - that's the beauty of working that close to the hardware). To better illustrate the memory problems that can occur, I've composed the following table containing the built-in SRAM, EEPROM, Flash and the architecture of
the most common Arduino boards.
<center>

|  Arduino Board          | EEPROM       |  SRAM   |    Flash    | Architecture |
|     :---:               | :---:        |  :---:  |    :---:    |     :---:    |
| Arduino UNO R3          | 1 KB         | 2 KB    |    32 KB    |     AVR      |
| Arduino MEGA 2560 R3    | 4 KB         | 8 KB    |   256 KB    |     AVR      |
| Arduino DUE             | NONE (IAP)   | 96 KB   |   512 KB    |     ARM      |
| Arduino Leonardo        | 1 KB         | 2.5 KB  |    32 KB    |     AVR      |
| Arduino YÚN             | 1 KB         | 2.5 KB  |    32 KB    |     AVR      |

</center>

### Playing around
Now that is clear that Arduino's SRAM is nowhere near enough for beefy projects, let's see how we can peak into it, optimize it and have fun with it. This may be obvious for some people, but just in case, when talking Arduino (especially the lower-end models with less than 4 KB of SRAM), you should avoid excessively using `malloc()` and `free()`. It's usually better (of course, if possible) to avoid dynamic memory allocation on Arduino because if the sketch is big enough, after enough allocations and deallocations, the memory will become fragmented and further allocations may fail because there is not enough memory available. Thing is, there might still be enough memory but not grouped in free chunk big enough for the allocation to proceed.

I know that this whole memory concept may sound confusing for the beginners, so I'll try to build a diagram that should be able to demonstrate how this phenomenon occurs.

<p align="center">
  <img src="https://raw.githubusercontent.com/GeoSn0w/geosn0w.github.io/master/images/memory%20segmentation%20on%20arduino.png"/>
</p>
  
