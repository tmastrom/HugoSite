+++
date = "2019-02-18"
title = "Real-Time Embedded System Project: Traffic Light System"
slug = "traffic-light"
+++



I worked in a team to simulate a traffic light system using **FreeRTOS** and an **STM32 Microcontroller** for my Real-Time Operating Systems course at the University of Victoria. [Source Code](https://github.com/alexandercote/RTOS_Traffic_Light_System)
{{< figure src="/images/rtos/go2.gif" caption="Traffic System in Action">}}
## Requirements 

Build a one-way traffic light system which simulates cars as green LEDs being lit up sequentially using a shift register IC. The traffic density will be determined by reading the Analog to Digital Converter (ADC) of the microcontroller. The ADC will be connected to a potentiometer. The traffic light times will change based on the traffic level. Heavier traffic corresponds to a longer green light and a shorter red light. When the light is green, traffic propagates along the entire length of 'road'. When the light turns yellow and then red, traffic before the stop line stops, traffic after the stop line continues to flow. New traffic will fill in gaps while waiting for the light.



## Implementation

Our program is implemented using cooperative scheduling, callback timers and mutex protected variables for deterministic behaviour. 

The program consists of four tasks:

**Traffic Flow Task** computes a discrete **flow** value based on the ADC reading. The ADC is connected to a 1k potentiometer

**Traffic Creator Task** creates a **car** based on the **flow** value. **car** is a boolean value (0 or 1)

**Traffic Light Task** adjusts the traffic light timers based on the **flow**. 

**Traffic Display Task** displays **cars** and the current traffic light using GPIO pins


{{< figure src="/images/rtos/off.jpg" caption="Hardware Setup" >}}

### Wiring Diagram
{{< figure src="/images/rtos/traffic_light_wiring.svg" caption="Wiring Diagram" >}}
### Parts List 

* STM32f4 
* Shift Register x2
* 1 kOhm Potentiometer
* 620 Ohm Resistor x19
* Green LED x17
* Yellow LED x1
* Red LED x1 
