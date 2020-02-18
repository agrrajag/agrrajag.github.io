---
title: TechnoTiki - Part 1
date: 2020-02-18 00:00:00
featured_image: '/images/technotiki-1/tiki.jpg'
excerpt: The High-Tech Lifestyle of a Low-Tech Hedgehog
---

![Photo of Tiki the hedgehog](/images/technotiki-1/tiki.jpg)

> !!! - Project In Progress - !!!

> The High-Tech Lifestyle of a Low-Tech Hedgehog

## The Issue
Tiki, our wonderful little hedgehog, has a fairly wide cage to run, explore, and tunnel around in. While her mom was out on a trip to Disney World, we moved one of the temperature sensors to be within camera view so we could monitor the temperature in the cage next to the food and water levels. I would be able to swing by and top off whatever she needed. We noticed that the temperature sensor on the opposite end of the cage was a few degrees lower than the probe controlling the heating lamp. 

Our idea was to set up some kind of system that could monitor the temperature in multiple locations in the cage and report it onto a dashboard we can access from our phones, a laptop, or a local screen close to the cage.

## The Plan

The plan included setting up a raspberry pi with multiple DS18B20 waterproof temperature sensors with 3-meters of wire. We would connect the temperature sensors to a breadboard and over to the Pi's GPIO input connections.

![Diagram showing circuit of the temperature sensors and the raspberry pi](/images/technotiki-1/sensor-diag.jpg)

*Source: <a href="https://www.raspberrypi.org/forums/viewtopic.php?t=167896">https://www.raspberrypi.org/forums/viewtopic.php?t=167896</a>*
 
 Since these are digital data feeds, we can place multiple sensors on the same one-wire input pin circuit. Power is sent to the temperature sensors using the 3.3v power pin to the power feeds on the sensors. For data, a 4.7k ohm resister is placed between the 3.3v power feeds and the data lines of the first sensor in the series. The rest of the data lines connect to that same feed without needing any additional resistors. The data line then connects back to the GPIO4 pin on the raspberry Pi. The ground lines go back to the ground pin on the Pi.

 From there, we would set up two applications on the Pi - one to collect and store the data, and the other to read the stored data and render a dashboard and calculations from the temperature array. Since the temperature sensors display in Celsius, we would want to run calculations to change this to Fahrenheit. We would also want to develop an average temperature readout of the cage between all of the sensors. Lastly, we would want to be able to store data over time so we can watch the efficiency of the heating, and control the heating lamp from the Pi in a future project.

## How We Did It

### Our Supply List
- <a href="https://www.amazon.com/gp/product/B07BC6WH7V">Raspberry Pi 3B+ board-only with power supply</a>
- <a href="https://www.amazon.com/gp/product/B06XWN9Q99">Samsung 32GB SDHC card</a>
- <a href="https://www.amazon.com/gp/product/B076V1XN3Q">3ple Decker Raspberry Pi Case </a>
- <a href="https://www.amazon.com/gp/product/B075SYJYN8">3ple Decker Half Breadboard Case with Breadboard</a>
- <a href="https://www.amazon.com/gp/product/B07PTYVPK3">4.7K Ohm Resisters
- <a href="https://www.amazon.com/gp/product/B01JP9BQSG">DS18B20 Temperature Probes with 3-meter cable
- <a href="https://www.amazon.com/gp/product/B07GS1Z3M3">Dupont Connector and Crimping Kit </a>

### Operating System Preparations
For those unfamiliar with Raspberry Pi's and loading up Raspbian Linux, you first have to flash the Raspbian image file onto the SD card before you are able to fire up the operating system. You can download the Raspbian image from the Raspberry Pi official site. There are a few options, including one with a GUI and one without. If you don't need the GUI front-end, the Lite version will require less resources to run your Pi.

<a href="https://www.raspberrypi.org/downloads/raspbian/" class="button button--small">Download Raspbian</a>

Once you have the image file downloaded, we need to use a flashing program to place it on the SD card. Balena Etcher is an easy program only requiring you to select the image file and SD card when inserted.

<a href="https://www.balena.io/etcher/" class="button button--small">Download Balena Etcher</a>


## Resources and References

<a href="https://knowledge.ryangarr.com/it-systems/internet-of-things-iot/raspberry-pi/projects/technotiki" class="button button--small">Check Out The Knowledge Guide</a>
