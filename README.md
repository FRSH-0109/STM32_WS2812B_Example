# STM32_WS2812B_Example
Used:
- STM32G030C8 microcontroller
- WS2812B addressable LEDs tape
- STM32CubeIDE (dedicated IDE for STM microcontrollers)

Projects uses my WS2812B LEDs driver library for controlling whole tape/strip of addressable LEDs. Certain protocol is achieved by controlling PWM via DMA mode. List of values prepared for every single diode is send by DMA-memory to peripheral mode.

WS2812B driver libs are written in C and are separated from STM32 HAL or other hardware dependent parts of project.
Library consists of:
 - ws2812b_driver.c
 - ws2812b_driver.h  

They can be found here: https://github.com/FRSH-0109/WS2812B_LED_Strip_Driver.git  
  
  
### How to use?  

**Get familiar with README file in the driver repository**  
It describes what should be changed in the driver header file.  
Driver .c and .h files have to be copied/imported to your project Src( .c) and Inc( .h) folders.  
https://github.com/FRSH-0109/WS2812B_LED_Strip_Driver.git  

**Configure hardware**  
Created function have to send proper PWM values, it is recommended to use DMA mode(memory to peripheral).  
![Screenshot from 2022-02-17 23-12-50](https://user-images.githubusercontent.com/64641846/154580604-1da37f4a-9f1d-471a-af23-a2c6d5979878.png)
> Selecting Timer, choosing clock source and channel mode(PWM generation).  
> Channel number is strongly connected with signal output pin.  

Commonly ws2812b diodeds are working with 
