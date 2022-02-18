# STM32_WS2812B_Example  
Projects uses my WS2812B LEDs driver library for controlling whole tape/strip of addressable LEDs. Certain protocol is achieved by controlling PWM via DMA mode. List of values prepared for every single diode is transfered as PWM signal of exact period time. It is expected that user knows how to configure new project and etc. in STM32CubeIDE.  

Used:
- STM32G030C8 microcontroller
- WS2812B addressable LEDs tape
- STM32CubeIDE (dedicated IDE for ST microcontrollers)  

WS2812B driver libs are written in C and are separated from STM32 HAL or other hardware dependent parts of project.
Library consists of:
 - ws2812b_driver.c
 - ws2812b_driver.h  

They can be found here: https://github.com/FRSH-0109/WS2812B_LED_Strip_Driver.git  
  
## How to use?  

- [Configure Hardware](#configure-hardware)
- [Prepare custom send function](#prepare-custom-send-function)
- [Usage example](#usage-example)  

----
### Get familiar with README file in the driver repository  
It describes what should be changed in the driver header file.  
Driver .c and .h files have to be copied/imported to your project Src( .c) and Inc( .h) folders.  
https://github.com/FRSH-0109/WS2812B_LED_Strip_Driver.git  

----
### Configure hardware

**Set MCU clock and timers clock frequnecy**  
In my project the frequency of mcu is set to 64MHz, as well as clock source for Timers(APB timer clocks)  
![Screenshot from 2022-02-17 23-13-41](https://user-images.githubusercontent.com/64641846/154681497-97a09647-e5f9-4a6e-b602-ea6cec0e1140.png)

**Created function have to send proper PWM values, it is recommended to use DMA mode(memory to peripheral).**

![Screenshot from 2022-02-17 23-12-50](https://user-images.githubusercontent.com/64641846/154580604-1da37f4a-9f1d-471a-af23-a2c6d5979878.png)
> Selecting Timer, choosing clock source and channel mode(PWM generation).  
> Channel number is strongly connected with signal output pin.

**Set proper PWM parameters**  
Prescalers is simply the divisor(divisor = value + 1, so written 0 sets prescaller to 1) between Timer APB clocks frequency and used in our Timer1 frequency. 64MHz / 1 = 64MHz on Timer1 clock.  
Counter Period is the value on which our counter will reload(start counting again from 0). It is set to 79(0-79 gives 80 counter steps). So our one timer cycle would count to 80 times on 64MHz freq, 64Mhz / 80 = 800000Hz = 800kHz = (1/800Khz)s = 0,00125ms = 1,25us(exactly what we would like to achieve with our specific diodes type).

![Screenshot from 2022-02-18 13-31-48](https://user-images.githubusercontent.com/64641846/154683480-325fb70a-a7c1-45a3-a258-edad872c5e50.png)

Make sure that your Timer channel polarity is set to high, every signal have to start by high volatge level.  
![Screenshot from 2022-02-18 13-37-12](https://user-images.githubusercontent.com/64641846/154684317-7d9a3ac0-dfd7-463f-908f-7ef8785542f5.png)

**Enable DMA mode for Timer**  
DMA have to be configured into memory to peripheral mode(array of values into PWM signal process).  
Timer data width: Half Word(16bit), beacuse we are using 16bit Timer1  
Memory data width: Byte(8bit), becuase array containing pwm data is uint8_t type variable  
Incerement adderess:	memory only  
![Screenshot from 2022-02-17 23-14-29](https://user-images.githubusercontent.com/64641846/154685102-de175573-2b2d-4cb0-8e49-acf0d3aa2a32.png)

![Screenshot from 2022-02-18 14-50-10](https://user-images.githubusercontent.com/64641846/154695091-91d10e76-2c02-4f73-9f02-80952be21c46.png)

----
### Prepare custom send function

To make whole driver library more hardware independent, send function have to be created by user.  
Only thing to do is to get the PWM data array and pass it to the DMA PWM pulse creating function.  
In case using the STM32 HAL, it will look like this:  
![Screenshot from 2022-02-18 17-11-45](https://user-images.githubusercontent.com/64641846/154720288-42b1a5b0-d9ab-4a92-b565-40acd8feeb81.png)
`ws2812bGetBytesArray(ledStrip);` - translates RGB data stored in ledStrip structure into pwm data, also stored in structure. This process is separeted into function, to let user make one translation after changing colors of many diodes.  
`HAL_TIM_PWM_Start_DMA(&htim1, TIM_CHANNEL_1, (uint32_t *)ledStrip->pwmData, ledStrip->bytesToSend);` - STM32 HAL function, which sends PWM via DMA. It needs few arguments like:
 - `(uint32_t *)ledStrip->pwmData` - pointer to begining of PWM data array
 - `ledStrip->bytesToSend` - how many bytes have to be send  

`ledStrip->dataSentFlag;` - this boolean variable prevents from sending data before previous one haven't been finished  

**Timer pulse finished interrupt**  
It is necessary to handle function which controls when data array has been send completely. I used HAL interrupt routine which is marked as `__weak` in default HAL files. After the last byte of data has been represented as PWM signal, interrupt changes `dataSentFlag`, so next send function can be executed successfully. Also the Timer PWM DMA sending routine is stopped, because normally it would work continiously.  
![Screenshot from 2022-02-18 17-08-25](https://user-images.githubusercontent.com/64641846/154721996-1d147025-e462-4abc-8ca3-2332ac2d607f.png)  

----
### Usage example

Setting red color on every two diodes. Delay can be removed  
![Screenshot from 2022-02-18 17-42-16](https://user-images.githubusercontent.com/64641846/154725502-14534026-98d9-4ba1-b2e3-246caa9b1c13.png)  

Creating wave effect on led tape(changable width). Every call of wave function generates next frame of it.  
![Screenshot from 2022-02-18 14-57-19](https://user-images.githubusercontent.com/64641846/154723866-fc7a148d-bee8-4f15-9bb7-7f10de8637da.png)  
