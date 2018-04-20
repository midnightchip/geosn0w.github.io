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
  
As you can see, multiple allocations and deallocations can result in free chunks in between allocated chunks. These free chunks can still be used if what you want to allocate is smaller or equal to the size of the free chunk. If it is bigger, the allocation will seek a bigger free chunk. In our diagram, that'd be in the right corner right after the 3rd allocated block, but if we allocate and deallocate a bit more, it will quickly build free chunks that may not be big enough therefore making the the allocation to fail.

### What can you do?
Depending on what kind of project you want to create, you may find the following good practices useful.
If you are working with strings (`char arrays`), you may wanna use the `F()` macro. To do that you transform your code from:
```C
setup(){
  Serial.print("Hello my name is George and I like to code.");
}
```

Into:

```c
setup(){
  Serial.print(F("Hello my name is George and I like to code."));
}
```
The `F()` macro pretty much puts your string on the Flash (code space) instead of putting it on the SRAM. The ammount of flash your Arduino has is bigger than the ammount of SRAM so that makes total sense. The way `F()` does that is quite complex but I will try to summarize it. I will take for this example the Arduino Mega 2560 which is `AVR` architecture. The `AVR` architecture is a Harvard Architecture and therefore, the `code` and the `data` are stored separately. Normally, when you call functions like `print()` or `Serial.print()` you pass a `char*` (Character Pointer) to them. This is pretty much the beginning (base) address of the character array holding your string on the flash. However, there is a problem. As you can see, `char*` is a pointer. In order to use whatever the pointer points to, you need to dereference it. When you dereference the pointer it will attempt to return the character stored in the SRAM (Data Space) instead of Flash (Code Space). What `F()` does is to pass a `__FlashStringHelper*` instead of a `char*`, this way pointing the function to the correct location (the code space).

This saves you quite a lot of memory. You can actually test this yourself. If you have an Arduino UNO, try creating 5-6 `character arrays` and hold a medium size sentence in each of them. You will see how much memory it chews. After that, use the `F()` macro and spot the difference.

