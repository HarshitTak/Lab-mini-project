# EE 690 Mini Project Report
### Group : 14
### Name : Harshit Tak and Miriyala Pranay Kamal
### ROLL NO. : EE23MT007 AND 200030033
### FACULTY ; Dr. ABHIJIT KSHIRSAGAR

## OBJECTIVE : 
 
Part 1:

Using only the Tiva board, make a stopwatch that will count the duration of a low-going pulse, and print the duration to a PC using the serial port.

Part 2:

Add some unique features to your implementation to make it stand out.


### Basic Working of Stopwatch
A stopwatch is a simple and commonly used timekeeping device designed to measure elapsed time. Its basic function is to start, stop, and reset the timing mechanism. Here's a breakdown of its primary functions:

#### Start:

Pressing the start button initiates the timing process. The stopwatch begins counting seconds, minutes, and sometimes hours, depending on its design.

#### Stop:

Pressing the stop button halts the timing process. This allows the user to freeze the elapsed time at a specific point.

#### Resume:

In many stopwatches, pressing the start button again after stopping will resume the timing from where it was paused.

#### Reset:

The reset button is used to zero the stopwatch, clearing the elapsed time and preparing it for a new timing session.

#### Split or Lap Time (Optional):

Some stopwatches have a feature that allows users to record split or lap times. A split time shows the elapsed time from the start, while the overall timer continues running. Lap time is similar but is measured from the last lap reset.

#### Countdown Timer (Optional):

Some stopwatches include a countdown timer function. Instead of measuring elapsed time, the user sets a specific time duration, and the stopwatch counts down to zero.

#### Display:

The stopwatch typically has a digital or analog display that shows the elapsed time in hours, minutes, and seconds. Some may also include additional features like date and alarm functions.

PWM is implemented using the inbuilt PWM peripheral.The peripherals allow us to generate square waveforms with a variable duty cycle.

## Functional Description 
The provided C code implements a basic stopwatch functionality using the TM4C123GH6PM microcontroller. It utilizes the SysTick timer for accurate timekeeping and GPIO Port F for controlling LEDs
and reading button inputs. The global variables c, min, sec, and mill track the current state of the stopwatch in terms of hours, minutes, seconds, and milliseconds. The SysTick_Handler function 
serves as the interrupt handler for the SysTick timer, updating the time values and controlling LED indicators based on the elapsed time. The prio function configures the priority of the GPIO
Port F interrupt, and the PortF_Handler function acts as an interrupt service routine, handling button presses to start, stop, or reset the stopwatch. The code is structured to run indefinitely
in a main loop, continuously updating the stopwatch based on timer interrupts and button inputs.
Initialy the use of LED was to indicate the increment in thre minute as the second reaches to 60 count value, but I also added the expression window through which i can display there respective values.

## What is SysTick?

Cortex-M4 includes an integrated system timer, SysTick, which provides a simple, 24-bit
clear-on-write, decrementing, wrap-on-zero counter with a flexible control mechanism which can
be configured using three registers, which are:

## SysTick Control and Status (STCTRL):
A control and status counter to configure its clock, enable the counter, enable the SysTick
interrupt, and determine counter status.

## SysTick Reload Value (STRELOAD):
The reload value for the counter, used to provide the counter's wrap value.

## SysTick Current Value (STCURRENT):
The current value of the counter.

## Code (Part 1)

#include <stdint.h>
#include <stdbool.h>
#include "tm4c123gh6pm.h"
#define SYST_CSR             (*((volatile uint32_t *)0xE000E010))
#define SYST_RVR             (*((volatile uint32_t *)0xE000E014))
#define SYST_CVR             (*((volatile uint32_t *)0xE000E018))
#define MASK_BITS 0x11
volatile int c=0,min=0,sec=0,mill=0;
void system_config(void)
{
    SYSCTL_RCGC2_R |= 0x00000020;;
    GPIO_PORTF_LOCK_R = 0x4C4F434B;
    GPIO_PORTF_CR_R = 0x01;
    GPIO_PORTF_DIR_R = 0x0E;
    GPIO_PORTF_DEN_R = 0x1F;
    GPIO_PORTF_PUR_R = 0x11;
}

void SysTick_Handler(void)
{
SYST_CSR&=~(1<<0);
if(mill<100)
{
    mill=mill+1;
    //GPIO_PORTF_DATA_R=0x08;
}
if(mill==100 && sec<60)
{
    mill=0;
    sec=sec+1;
    GPIO_PORTF_DATA_R=0x04;

}
if(mill==100 && sec==60 && min<60)
{
    mill=0;
    sec=0;
    min=min+1;
    GPIO_PORTF_DATA_R=0x02;

}
if(mill==100 && sec==60 && min==60)
{
    mill=0;
    sec=0;
    min=0;
    GPIO_PORTF_DATA_R=0x0E;
    }
SYST_CSR=0x07;
}


void prio(void)
{
    GPIO_PORTF_DATA_R&=0x00;
    GPIO_PORTF_IM_R &= ~MASK_BITS; // mask interrupt by clearing bits
    GPIO_PORTF_IS_R &= ~MASK_BITS; // edge sensitive interrupts
    GPIO_PORTF_IBE_R &= ~MASK_BITS; // interrupt NOT on both edges
    GPIO_PORTF_IEV_R &= ~MASK_BITS;

    NVIC_PRI7_R = (NVIC_PRI7_R & 0xF1FFFFFF) | (7 << 21);
    NVIC_EN0_R |= 1 << 30;
    GPIO_PORTF_ICR_R = MASK_BITS;
    GPIO_PORTF_IM_R |= MASK_BITS;
    NVIC_SYS_PRI3_R = (NVIC_SYS_PRI3_R & 0x1FFFFFFF) | (3 << 29) ;
}

void portF_handler()
{ int k=0,i = 100;
    k=GPIO_PORTF_MIS_R;
    GPIO_PORTF_IM_R &= ~(0x11);
    if(k & 0X01)
    {
        SYST_CSR =0x07;
    }
    if(k & 0X10)
    {

                                        if(c==0){SYST_CSR&=~(1<<0);GPIO_PORTF_DATA_R=0x00;}
                                        if(c==1){SYST_CSR&=~(1<<0);SYST_CVR==0x00;GPIO_PORTF_DATA_R=0x00;min=0;sec=0;mill=0;}
                                        c++;
                                        if(c==2){c=0;}
    }
    while(i--);
    GPIO_PORTF_IM_R |= 0x11;
    GPIO_PORTF_ICR_R |= 0x11;

}
void main(void)
{
    SYST_RVR=160000-1;
    SYST_CVR==0x00;
    system_config();
    prio();
    while(1){};

}

## Observation 

The presented C code orchestrates a fundamental stopwatch mechanism on the TM4C123GH6PM microcontroller, leveraging the SysTick timer for precise time tracking and GPIO Port F to govern LED displays 
and interpret button inputs. Essential global variables such as c, min, sec, and mill dynamically capture the ongoing state of the stopwatch, meticulously measuring hours, minutes, and seconds. 
The code orchestrates a cohesive integration of hardware elements: LEDs serve as visual indicators to convey the running, stopped, and reset states of the stopwatch, while button inputs on GPIO 
Port F are strategically harnessed to initiate, halt, and reset the stopwatch. This implementation provides a robust foundation for building interactive timekeeping applications on the microcontroller
platform.

## Output :
### Output for 10.57 Mobile Stopwatch<br>

![Output for 10](https://github.com/HarshitTak/Lab-mini-project/blob/main/stop%20watch/10.57.jpg)

### Output for 10.45 Stopwatch<br>

![Output for 10](https://github.com/HarshitTak/Lab-mini-project/blob/main/stop%20watch/10.45.png)

### Output for 01.17.41 Mobile Stopwatch<br>

![Output for 1 min](https://github.com/HarshitTak/Lab-mini-project/blob/main/stop%20watch/1.17.41.jpg)

 ### Output for 01.17.28 Stopwatch<br>

![Output for 10](https://github.com/HarshitTak/Lab-mini-project/blob/main/stop%20watch/1.17.28.png)




