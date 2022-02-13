# STM32_WS2812B_Example
Used:
- STM32G030C8
- WS2812B addressable LEDs tape

Projects uses my WS2812B LEDs driver library for controlling whole tape/strip of addressable LEDs. Certain protocol is achieved by controlling PWM via DMA mode. List of values prepared for every single diode is used by DMA memory to peripheral mode.

WS2812B driver libs are written in C and are separated from STM32 HAL or other hardware dependent parts of project.
Library consists of:
 - ws2812b_driver.c
 - ws2812b_driver.h
They can be found here: https://github.com/FRSH-0109/WS2812B_LED_Strip_Driver.git
