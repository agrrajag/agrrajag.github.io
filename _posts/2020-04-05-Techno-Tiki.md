---
title: TechnoTiki - Part 1
date: 2020-04-05 00:00:00
featured_image: '/images/technotiki-1/tiki.jpg'
excerpt: The High-Tech Lifestyle of a Low-Tech Hedgehog
---

![Photo of Tiki the hedgehog](/images/technotiki-1/tiki.jpg)

> !!! - Project In Progress - !!!

> The High-Tech Lifestyle of a Low-Tech Hedgehog

## The Issue
Tiki, the wonderfully sassy little hedgehog this project is named after, passed away recently after some medical complications. While her spunky little self was with us, my fiancée and I were brainstorming ideas on how to make her home more comfortable and include monitoring data-points to ensure we have a happy hedgehog. With a new hedgehog - who has yet to be named - on the way, we decided it was fitting to continue our project and name it after our sweet and sassy little Tiki...

## The Project

TechnoTiki is planned to have multiple phases:

### Phase 1 
This will look at temperature logging from multiple probes inside the cage. With our original hedgehog setup, our visual temperature sensor and heating element probe did not align the best and could vary by multiple degrees. This phase will give us a dashboard we can log into remotely on our phones or laptops to see the temperatures in three to five different spots in the cage as well as the average cage temperature of all the probes.
### Phase 2 
This will find a way to log her activity on the wheel. I am still brainstorming exactly what we want this to look like - if it is yards-per-day or total miles. There have been other projects that have also posted these statistics to Twitter. We will see what this looks like in the future

### Phase 3
This will look at bringing automation into the mix by connecting the heating element to the Pi to help keep the average cage temperature aligned. Not sure how we will do this quite yet.

## Phase 1 - The Plan

The plan included setting up a raspberry pi with multiple DS18B20 waterproof temperature sensors with 3-meters of wire. We would connect the temperature sensors to a breadboard and over to the Pi's GPIO input connections.

![Diagram showing circuit of the temperature sensors and the raspberry pi](/images/technotiki-1/sensor-diag.jpg)

*Source: <a href="https://www.raspberrypi.org/forums/viewtopic.php?t=167896">https://www.raspberrypi.org/forums/viewtopic.php?t=167896</a>*
 
 Since these are digital data feeds, we can place multiple sensors on the same one-wire input pin circuit. Power is sent to the temperature sensors using the 5v power pin to the power feeds on the sensors. For data, a 4.7k ohm resister is placed between the 5v power feeds and the data lines of the first sensor in the series. The rest of the data lines connect to that same feed without needing any additional resistors. The data line then connects back to the GPIO4 pin on the raspberry Pi. The ground lines go back to the ground pin on the Pi.

 I originally mentioned connecting the sensor voltage wire to the 3.3v pin, however, this appeared to not be enough voltage with the amount of wire we had being used. Switching to the 5v allowed data to pass.

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

#### Raspberry Pi Imager
This documentation originally suggested downloading Raspbian and then Balena etcher to flash the SD card. Since the original draft of this document, the curators of Raspberry Pi and Raspbian have released a program called the Raspberry Pi Imager which does all of the work needed for Balena for you. You just need to download the imager, select which Operating System you would like (either full or lite will work), the SD card you would like to use, then select flash. After a few minutes, it will cache the most current version of Raspbian and prepare the SD card for you.

<a href="https://www.raspberrypi.org/downloads/" class="button button--small">Download Raspberry Pi Imager</a>

#### Raspbian
*This documentation is no longer needed, but good to have in case you want to go this route instead of the Raspberry Pi Imager*

For those unfamiliar with Raspberry Pi's and loading up Raspbian Linux, you first have to flash the Raspbian image file onto the SD card before you are able to fire up the operating system. You can download the Raspbian image from the Raspberry Pi official site. There are a few options, including one with a GUI and one without. If you don't need the GUI front-end, the Lite version will require less resources to run your Pi.

<a href="https://www.raspberrypi.org/downloads/raspbian/" class="button button--small">Download Raspbian</a>

#### Balena
*This documentation is no longer needed, but good to have in case you want to go this route instead of the Raspberry Pi Imager*

Once you have the image file downloaded, we need to use a flashing program to place it on the SD card. Balena Etcher is an easy program only requiring you to select the image file and SD card when inserted.

<a href="https://www.balena.io/etcher/" class="button button--small">Download Balena Etcher</a>

![Screenshot of Balena program showing settings to flash the SD card](/images/technotiki-1/balena.PNG)

#### Raspi-Config and System Updates

We have the SD card flashed and the Raspberry Pi loaded up in its case. Let's put the two together, test the Pi to make sure it is not dead-on-arrival, connect it to the network to run updates, and update some configuration options.

~~~
sudo raspi-config
~~~
- Set Password
- Set Network Options (WiFi if needed)
- Set Localization Options
    * EN-US UTF-8
    * US timezone
    * US WiFi frequency Channels
    * Appropriate Keyboard Layout
- Set Interfacing Options
    * Enable SSH
    * Enable 1-Wire

Once you have everything set inside of raspi-config, you can select Finish. We will then move to updating our software on the Pi.

~~~
sudo apt update
sudo apt upgrade -y
~~~

### Breadboard Preparations
*This documentation is out of date*
![Diagram showing breadboard circuit of the temperature sensors, breadboard, and the raspberry pi](/images/technotiki-1/breadboard.png)

### Testing the Sensors
Here is where the project gets a little tricky and took me about a day to troubleshoot. 

Two things to note:
1. Many of the blogs referencing these types of connectors suggested running modprobe. In the latest Raspbian image, you do NOT need to run this. Simply enabling the 1-wire interface in raspi-config will enable everything needed. I did not need to modify anything in /etc/modules or /boot/config.

2. When we go to test the data, you want to see folder names starting with 28-. If you see 00-, you have bad data. In my situation, we had bad wires connecting to the breadboard. I had to put connectors on a new wire and it started working.

To test the sensors:

~~~
cd /sys/bus/w1/devices
ls
~~~

In here, you should find one or multiple folders starting with 28-. This means the Pi has registered the probe and the number after the dash is the serial number of the probe. Now, let's take a look at the data being received. Change the xxxxx to the serial of the probe you want to connect to.

~~~
cd 28-xxxxxxxxxxxx 
cat w1_slave
~~~

On the second line, you should see t=xxxxx. This is the temperature in Celsius. For mine at the time of writing, mine showed t=24500, meaning it is 24.500 degrees C, or 76.1 degrees F.


## Resources and References

<a href="https://knowledge.ryangarr.com/it-systems/internet-of-things-iot/raspberry-pi/projects/technotiki" class="button button--small">Check Out The Knowledge Guide</a>

Here are some guides I used to help come up with our end product:
- <a href="https://www.circuits.dk/temperature-logger-running-on-raspberry-pi/">https://www.circuits.dk/temperature-logger-running-on-raspberry-pi/</a>
- <a href="https://www.definit.co.uk/2018/07/monitoring-temperature-and-humidity-with-a-raspberry-pi-3-dht22-sensor-influxdb-and-grafana/">https://www.definit.co.uk/2018/07/monitoring-temperature-and-humidity-with-a-raspberry-pi-3-dht22-sensor-influxdb-and-grafana/</a>
- <a href="https://www.terminalbytes.com/temperature-using-raspberry-pi-grafana/">https://www.terminalbytes.com/temperature-using-raspberry-pi-grafana/</a>
- <a href="http://www.hietala.org/temperature-reading-influxdb.html">http://www.hietala.org/temperature-reading-influxdb.html</a>
- <a href="https://ainsey11.com/2019/04/07/raspberry-pi-temperature-sensor-that-logs-to-grafana/">https://ainsey11.com/2019/04/07/raspberry-pi-temperature-sensor-that-logs-to-grafana/</a>