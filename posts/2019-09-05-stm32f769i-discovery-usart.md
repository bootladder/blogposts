---
layout: post
title: STM32F769 Discovery USART
---
When working on embedded systems, a UART is mandatory.  The codebase I'm starting with
did not include application level UART code.  
So I think getting the UART to work is a good first task.  

# USART Driver and App code:  The General Idea
In super simple embedded systems, driving the UART can be as simple as calling Init() followed by read() or write().  
In complicated ones, there's a mess of clock configuration, UART register configuration, pin multiplexing, etc.  
But regardless, from a software standpoint, you call Init() somewhere in main() before the while(1),
and then you call write() as you please.  
  
Let's see how main() works in this codebase I have.  Recall this is the STemWin demo in the STM32Cube_FW_F7_V1.15.0.  
  
I like how it's organized.  Basically it goes  
```
CPU Init
HAL Init
BSP Init
Kernel Init
Start Scheduler
```
Each step going "higher" or "outside" .  Cool!  
  
Considering the UART is on chip and there is no relevant hardware on board, this is at HAL level.  
  
Inside the HAL_Init() of this project, I'd say that's CPU level stuff.  The important thing it does is
initialize the 1ms tick for FreeRTOS.  
So actually the HAL is all initialiezd inside the BSP inits.  For example, BSP_LED_Init() makes calls
to HAL_GPIO_Init().  
  
So I'll go ahead and stick a USART_Init() right in main().  
  
How much help can I get with this?  Let's do some grepping for UART stuff.
Right off the bat I see there is both a USART and a UART.  Wow.  
And I don't see any demo code using USART or UART.  
Looking at the schematic for the discovery kit, indeed there are signals eg. USART6_TX and UART5_Tx.  
The signals I need are UART5_Tx and UART5_Rx, on pins PD2 and PC12 respectively.  
The reference manual only shows a peripheral for USART.  So it must be that UART refers to pins that are
only usable in asynchronous modes but still use the USART peripheral.  
But nonetheless, the STM32Cube drivers have both a stm32f7xx_hal_uart.c and a stm32f7xx_hal_usart.c.
I'll go with the uart.c.  
  
  
A few hours later... I have a working UART, from the Examples/ directory in the Cube FW codebase.  
  
