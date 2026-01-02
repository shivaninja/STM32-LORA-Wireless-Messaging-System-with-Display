# STM32-LORA-Wireless-Messaging-System-with-Display
STM32 + LoRA Text Transmission

This repository contains the source code and documentation for a text send & receive system using STM32 microcontrollers and Ebyte 915 MHz UART LoRA modules. 
## Overview

Using two STM32 boards (e.g., Blue Pill or similar) and E220-900T22D LoRA modules, this project demonstrates bidirectional text transmission over long range (up to several hundred meters) via UART communication. 
The system allows two STM32 boards to communicate wirelessly using simple UART functions without needing additional LoRA libraries. 
I have used two sets of STM32 and Ebyte 915MHz UART LORA Module to transmit and receive data between long distance. This LORA module does not need library. We can simply use HAL_UART_Transmit() and HAL_UART_Receive() to transmit and receive. When receiving we need to decode the data from character to integer. Manufacturer sets Channel, Address and key to default value so no need to change these until we complete basic circuit and do a distance testing. It will work out of the box. Make sure to use both modules with same part number. E220-900T22D module is rated for 5km distance. Because I used low voltage of 3.3V VCC, I was able to get about 500m (half km) easily without line of sight using about 25mW power.


The LoRA modules are configured with default channel, address, and key settings — simplifying setup and enabling immediate transmission and reception. 

```
What’s Included
|-- /Core/Inc
│    |── fonts.h
│    |── ssd1306.h
|-- /Core/Src
│    ├── fonts.c
│    |── ssd1306.c
|-- main.c
|-- .gitignore
|-- README.md
|-- STM32CubeIDE Project Files
```

Replace filenames/structure if your project differs.

## Hardware Requirements
Component	Quantity

STM32 Microcontroller Board (e.g., STM32F103C8T6)	2

Ebyte E220-900T22D LoRA UART Modules	2

OLED Display (optional for debugging)	1

Wires, Power Supply (3.3V)	As needed

## Software Requirements

STM32CubeIDE

Basic familiarity with STM32 HAL functions

Understanding of UART transmit and receive functions

## Setup & Wiring

Connect STM32 UART1 TX/RX to the LoRA module RX/TX respectively (cross-connected).

Provide 3.3V VCC and GND to both the LoRA module and STM32 board.

(Optional) Connect an OLED display via I2C for visual feedback.

Ensure both LoRA modules are the same model and set for the same frequency/parameters. 


## STM32CubeIDE Settings
1. ADC1 - IN2 (tick)

2. Parameter Settings --> ADC Settings --> Continuous Conversion Mode (Enabled)

3. Set PA1 to GPIO_EXTI1

4. Set PA3 to GPIO_EXTI3

5. Click NVIC → EXTI line1 interrupt → Enabled (Tick)

6. Click NVIC → EXTI line3 interrupt → Enabled (Tick)

7. Click GPIO → Select PA1 → GPIO Pull-up/Pull-down → Pull-down

8. Click GPIO → Select PA3 → GPIO Pull-up/Pull-down → Pull-down

9. Enable USART1 asynchronous

10. Parameter Settings --> Basic Parameters --> Baud rate 9600

11. NVIC Settings --> USART1 global interrupt --> (Tick)

12. Click connectivity --> Click I2C1

13. For I2C select I2C

14. Configuration --> Parameter Settings
    
## Key Project Features
## LoRA Communication

Uses simple UART functions:

HAL_UART_Transmit();
HAL_UART_Receive();


No additional LoRA libraries required — full communication handled via UART. 
micropeta.com

## Encoding

Received data is decoded from characters to integers for processing on the receiving STM32. 
micropeta.com

## OLED Display Output

Optionally displays transmitted and received text on an SSD1306 OLED screen using the included fonts and driver. 
micropeta.com

## Code Snippet
```
Here’s the main communication loop:

HAL_ADC_Start(&hadc1);
HAL_ADC_PollForConversion(&hadc1,1000);
readValue = HAL_ADC_GetValue(&hadc1);
HAL_ADC_Stop(&hadc1);

// Convert ADC value to char
charValue = readValue / 65 + 32;
singleChar[0] = charValue;

// Display received text
SSD1306_GotoXY(0, 0);
SSD1306_Puts((char *)rxBuffer, &Font_11x18, 1);

// Display current TX char
SSD1306_GotoXY(53, 37);
SSD1306_Puts((char *)singleChar, &Font_16x26, 1);
SSD1306_UpdateScreen();
HAL_Delay(50);


Interrupt routine for UART reception:

void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
  if (huart->Instance == USART1) {
    if (rxData != 13) {
      rxBuffer[rxIndex++] = rxData;
    } else {
      // Reset buffer once CR received
      rxIndex = 0;
      memset(rxBuffer, 0, sizeof(rxBuffer));
    }
    HAL_UART_Receive_IT(&huart1, &rxData, 1);
  }
}
```
# Notes & Tips

The LoRA modules used (E220-900T22D) are rated up to 5 km LOS, though power and frequency regulations vary by region. Always check local radio regulations. 
micropeta.com

For in-house testing without antennas, be mindful that the modules can overheat due to RF power reflection; power cycling may be needed. 
