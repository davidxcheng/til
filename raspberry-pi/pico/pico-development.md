# Pico Development - The Hard Way

The easy way to develop code for a Pico board is to use `VS Code` and the `Raspberry Pi Pico` plugin. Or, if you are using `Raspberry Pi OS` and want to write `Python`, you can use the `Thonny` IDE. Both these options lets you focus on writing the code and takes care of compiling it and flashing it to your Pico board. But part of my reason for buying a Pico was to get a little bit familiar with a microcontroller so I opted for a less "magical" setup - and this document contains my learnings. Some of the information comes from `Chat GPT` answers.

## What is a Pico?

The `Raspberry Pi Pico 2` is a microcontroller which means that it's not a full-fledged computer like a Raspberry Pi 4. It does not have an operating system. Instead, it runs code directly on its RP2350 chip which have the following specifications:

- Processor: Dual-core ARM Cortex-M0+ (up to 150 MHz)
- Memory: 520 KB SRAM
- Flash Storage: Uses external QSPI Flash (default 2 MB on Pico)
- Power: Runs at 1.8V to 3.3V, but needs 5V for USB

> Note: The first `Raspberry Pi Pico` was built on the `RP2040` chip and that name is still present in some software.

A brand-new Raspberry Pi Pico is completely blank, it does not have any firmware or runtime pre-installed. However, it does have a small ROM-based USB bootloader (called RP2040 BOOTSEL mode). This allows it to appear as a USB storage device when you hold the BOOTSEL button while plugging it into your computer. And that is the easiest/most common way to deploy code to the Pico:

1. Connect it to a USB port while holding down the BOOTSEL button
2. Write your ELF file to the flash storage by copying it to the "storage device"

## How to write code for the Pico?

The `Pico SDK` compiles C/C++ for the RP2040. It uses `CMAKE` to build the project and the output is a UF2, ELF or BIN file that is flashed to the Pico and runs bare-metal (no OS needed) on the RP2350.

This is an example of the project layout:

```
|-- pico-sdk
|-- picotool
|-- my_project/
  │── CMakeLists.txt        # Main CMake configuration
  │── main.cpp              # Your main C++ code
  │── include/              # (Optional) Header files
  │── src/                  # (Optional) Other source files
  │── build/                # (Generated build files go here)
```

## My setup

The goal of my little project was to connect an air qualtiy sensor to the GPIO pins of a `Raspberry Pi Pico 2 WH`. The `W` stands for "wireless" and means that it has a chip with wifi and bluetooth. For the `Pico 2` the microcontroller chip has been updated from `RP2040` to `RP2350`.

Since the Pico doesn't have an OS you have to write and compile the code on a different machine. I chose to do it on a `Raspberry Pi 4` running `Raspberry Pi OS`. The `Raspberry Pi 4` has it's own set of GPIO pins but I wanted to dabble with a microcontroller, hence the extra hardware. I could also have used my MBP to write and compile the code but didn't want the `Pico` board dangling from the laptop I use every day.

## Installing CMAKE

`CMAKE` is a cross-platform tool that generates `make` files. In other words a layer on top of `make`.

I downloaded the [`cmake-4.0.2-linux-aarch64.sh` file](https://cmake.org/download/), which is a self extracting gziped tar file, and installed under `/usr/local/bin`. It was a little bit tricky (for a linux newbie) to accept the license but by running `sh cmake-4.0.2-linux-aarch64.sh --help` you got a list of options and one of them was to automatically accept the license.

EDIT: 2nd time I installed it at `/usr/share/cmake/cmake-4.0.2-linux-aarch64/`

## Installing the Pico SDK

~~Installning the Pico SDK is just a matter of downloading `pico-sdk-2.1.1.tar.gz` and unzipping it. Set `PICO_SDK_PATH` or pass `-DPICO_SDK_PATH=` to cmake.~~

Unzipping `pico-sdk-2.1.1.tar.gz` will not work if it's a pico w board and you wish to use any wireless feature. The pico git repository uses git modules and the tar file does not include the repositories of the submodules. Instead, `git clone` the [Raspberry Pi Pico SDK repository](https://github.com/raspberrypi/pico-sdk.git) and then run `git submodule update --init`. This will clone code into the `lib` directory.

### SDK dependencies

The SDK depends on a cross-compiler to compile for the `RP2350` chip so I installed `gcc-arm-none-eabi` via `apt` (`sudo apt install gcc-arm-none-eabi`).

The SDK can produce the ELF file but in order to flash the ELF file to the pico it has to be wrapped as an UF2 file and this is done using `picotool`. From page 11 in the [Pico C SDK docs](https://datasheets.raspberrypi.com/pico/raspberry-pi-pico-c-sdk.pdf):

> The ELF file is converted to a UF2 using picotool.

I installed `picotool` by cloning the git repo and then following the instructions to build it.

The `Pico SDK` contains `pico_sdk_import.cmake` that should be copied to the root of your project directory and then you can create the `make` files by running this command from the `build` directory:

> /usr/share/cmake/cmake-4.0.2-linux-aarch64/bin/cmake -DPICO_BOARD=pico2_w -DPICO_SDK_PATH=/home/dxc/pico/pico-sdk ..

And then comile the code:

> make -j4

The compiled binaries show up in the `build` directory.

> cp my_program.uf2 /media/dxc/RP2350/

## Random

I/O Interfaces:

- I2C, SPI, UART (multiple instances of each)
- 12-bit ADC (3 channels + internal temperature sensor)
- PIO (Programmable I/O) for custom peripherals
- USB: Full-speed USB 1.1 (host & device mode)

2025-10-19: Trying to build the SGP40 VOC sensor example for Pico