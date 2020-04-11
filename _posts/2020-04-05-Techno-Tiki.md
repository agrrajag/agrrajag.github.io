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

![Screenshot of Raspbery Pi Imager program](/images/technotiki-1/pi-imager.png)

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

### InfluxDB
#### Installation
~~~
cd /home/pi/
wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
echo "deb https://repos.influxdata.com/debian buster stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
sudo apt update
sudo apt install influxdb
sudo systemctl unmask influxdb
sudo systemctl enable influxdb
sudo systemctl start influxdb
~~~

**Install Influx python libraries**
~~~
python3 -m pip install influxdb
~~~

#### Configuration
Create the database and use show databases to confirm creation
~~~
CREATE DATABASE temp_logger_db
SHOW DATABASES
exit
~~~

Create the script to poll the temperature data
~~~
cd /home/pi
nano temperature.py
~~~

Copy and paste your temperature python script. We have used a modified version of the one Terje posted on Circuits.dk site.

<a href="https://www.circuits.dk/temperature-logger-running-on-raspberry-pi/" class="button button--small">Circuits.dk - "Temperature logger running on Raspberry Pi"</a>

~~~
# -*- coding: utf-8 -*-
import os
import glob
import argparse
import time
import datetime
import sys
from influxdb import InfluxDBClient

os.system('modprobe w1-gpio')
os.system('modprobe w1-therm')

# add more sensor variables here based on your setup

temp=['sensor code','tttttttttt','ddddddddddd','ssssssssss']
base_dir = '/sys/bus/w1/devices/'

device_folders = glob.glob(base_dir + '28*')

snum=3 #Number of connected temperature sensors

# Set required InfluxDB parameters.
# (this could be added to the program args instead of beeing hard coded...)
host = "localhost" #Could also use local ip address like "192.168.1.136"
port = 8086
user = "root"
password = "root"
 
# Sample period (s).
# How frequently we will write sensor data from the temperature sensors to the database.
sampling_period = 10

def read_temp_raw(device_file): 
    f = open(device_file, 'r')
    lines = f.readlines()
    f.close()
    return lines
 
def read_temp(device_file): # checks the temp recieved for errors
    lines = read_temp_raw(device_file)
    while lines[0].strip()[-3:] != 'YES':
        time.sleep(0.2)
        lines = read_temp_raw(device_file)

    equals_pos = lines[1].find('t=')
    if equals_pos != -1:
        temp_string = lines[1][equals_pos+2:]
        # set proper decimal place for C
        temp = float(temp_string) / 1000.0
        # Round temp to 2 decimal points
        temp = round(temp, 1)
    # value of temp might be unknown here if equals_pos == -1
    return temp

def get_args():
    '''This function parses and returns arguments passed in'''
    # Assign description to the help doc
    parser = argparse.ArgumentParser(description='Program writes measurements data from the connected DS18B20 to specified influx db.')
    # Add arguments
    parser.add_argument(
        '-db','--database', type=str, help='Database name', required=True)
    parser.add_argument(
        '-sn','--session', type=str, help='Session', required=True)
    now = datetime.datetime.now()
    parser.add_argument(
        '-rn','--run', type=str, help='Run number', required=False,default=now.strftime("%Y%m%d%H%M"))
    
    # Array of all arguments passed to script
    args=parser.parse_args()
    # Assign args to variables
    dbname=args.database
    runNo=args.run
    session=args.session
    return dbname, session,runNo
    
def get_data_points():
    # Get the three measurement values from the DS18B20 sensors
    for sensors in range (snum): # change number of sensors based on your setup
        device_file=device_folders[sensors]+ '/w1_slave'
        temp[sensors] = read_temp(device_file)
        print (device_file,sensors,temp[sensors])
    # Get a local timestamp
    timestamp=datetime.datetime.utcnow().isoformat()
    
    # Create Influxdb datapoints (using lineprotocol as of Influxdb >1.1)
    datapoints = [
        {
            "measurement": session,
            "tags": {"runNum": runNo,},
            "time": timestamp,
            "fields": {"temperature 1":temp[0],"temperature 2":temp[1],"temperature 3":temp[2],}
        }
        ]
    return datapoints

# Match return values from get_arguments()
# and assign to their respective variables
dbname, session, runNo =get_args()   
print ("Session: ", session)
print ("Run No: ", runNo)
print ("DB name: ", dbname)

# Initialize the Influxdb client
client = InfluxDBClient(host, port, user, password, dbname)
        
try:
     while True:
        # Write datapoints to InfluxDB
        datapoints=get_data_points()
        bResult=client.write_points(datapoints)
        print("Write points {0} Bresult:{1}".format(datapoints,bResult))
            
        # Wait for next sample
        time.sleep(sampling_period)
        
        # Run until keyboard ctrl-c
except KeyboardInterrupt:
    print ("Program stopped by keyboard interrupt [CTRL_C] by user. ")
~~~

#### Start temporary data collection

**This is currently being worked on. I am having issues finding a way to have the Raspberry Pi automatically start the script at reboot. Cronjobs did not seem to work. For now, we can use a manual trigger to get the program running. The following code will send the data to a measurement named test1. You can go back later and modify Influx and Grafana to use another shard after you do your testing.**

If connected through VNC and you want to view the raw data going into the database:
~~~
python3 /home/pi/templogger.py -db=temp_logger_db -sn=test1
~~~

If you are SSH and want the script to persist outside your session:
~~~
nohup python3 /home/pi/templogger.py -db=temp_logger_db -sn=test1
~~~

### Grafana
#### Installation
~~~
sudo apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/oss/release/grafana_6.7.2_armhf.deb
sudo dpkg -i grafana_6.7.2_armhf.deb
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable grafana-server
sudo /bin/systemctl start grafana-server
~~~

We are going to go ahead and install the plugin Blendstat on Grafana which beautifully creates an average of all the different datapoints

~~~
sudo grafana-cli plugins install farski-blendstat-panel
~~~

#### Configuration
You should be able to test your connection to the server by opening a browser and going to the IP of your Pi port 3000. We will spend time later creating a reverse proxy over HTTPS and locking down this insecure port.

##### Setting Up The Dashboard

###### Line Graph

- Log in to Grafana
- Select the + button
- Select Add Query
- Select "select measurement" and change to test1
- Where it says field(value), select the word "value"
- Change this to the temperature probe you want to monitor
- Select the + button to the right of field and choose Selectors > last
- Select the + button and select Math -> math
- In math, change the formula to "*1.8+32"
- Change Group By to be time(10s), fill(linear)
- Change Alias to Probe 1
- Select the duplicate button at the top of that section and follow these steps for probes 2 and 3
- After you have added the queries, go on the left to the graph icon for Visualization
- Change draw mode to lines, Left Y Unit to Temperature -> Fahrenheit. You can also use this screen to add thresholds.
- When complete, hit the <- button at the top left of the screen

**More details to come about setting up the graphs**

## Cyber Security and Online Access
Since this is a tech blog, we also have to mention the security aspect of this project. For the previous aspects of the project, this project was all managed internally on the local network. The default configuration of a raspberry pi has the firewall (ufw) disabled, and Grafana using HTTP as its default protocol. As I am learning cybersecurity and how to effectively deploy tools in my home and in the workplace, it's always good to practice your techniques.

### Reverse NGINX HTTPS proxy
Grafana by default runs HTTP on port 3000. This means you have an unencrypted session that requires a login (viewable in plaintext to a network sniffer) when you are using the service. We can set up NGINX to host an encrypted session between our computer and the raspberry pi. This will allow us to use https://hostname.domain.tld instead of http://hostname.domain.tld:3000. For this section of the project, we used a guide provided by the wonderful people over at DigitalOcean:

<a href="https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-with-ssl-as-a-reverse-proxy-for-jenkins" class="button button--small">Digital Ocean - "How To Configure Nginx with SSL as a Reverse Proxy for Jenkins"</a>

**Note: HTTP 301 redirect is broken in this current draft. I believe it is due to the lack of the hostname in our DNS and using the IP address instead. More troubleshooting should resolve this.**

#### Installation

First, you are going to want to install NGINX and create your own certificate and key. While a self-signed certificate is not best practice, it is helpful to deploy an HTTPS server quickly. I plan to come back and update this once I figure out Let's Encrypt and the automation behind that. To switch from a self-signed to a trusted certificate, you would just need to update /etc/nginx/cert.key with the private key, and /etc/nginx/cert.crt with the trusted certificate chain.

~~~
sudo apt update
sudo apt install nginx
cd /etc/nginx
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/cert.key -out /etc/nginx/cert.crt
~~~

Here, it will ask you questions to fill out the information for the certificate. Please keep an eye for where it asks you to abbreviate and where it asks you to spell out locations. The most important part of this section is the common name needs to be the fully qualified domain name (FQDN) of the service you are trying to connect to (dashboard.example.com)

From here, we are going to modify the NGINX configuration to set up our reverse proxy.

~~~
cd /etc/nginx/sites-available/
sudo cp default default-backup
sudo nano /etc/nginx/sites-available/default
~~~

Paste and modify the following script to what you need:

~~~
server {
    listen 80;
    return 301 https://$host$request_uri;
}

server {

    listen 443;
    server_name changeme.example.com;

    ssl_certificate           /etc/nginx/cert.crt;
    ssl_certificate_key       /etc/nginx/cert.key;

    ssl on;
    ssl_session_cache  builtin:1000  shared:SSL:10m;
    ssl_protocols  TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;
    ssl_prefer_server_ciphers on;

    access_log            /var/log/nginx/changeme.access.log;

    location / {

      proxy_set_header        Host $host;
      proxy_set_header        X-Real-IP $remote_addr;
      proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header        X-Forwarded-Proto $scheme;

      # Fix the “It appears that your reverse proxy set up is broken" error.
      proxy_pass          http://localhost:3000;
      proxy_read_timeout  90;

      proxy_redirect      http://localhost:3000 https://changeme.example.com;
    }
  }

~~~


## Resources and References

<a href="https://knowledge.ryangarr.com/it-systems/internet-of-things-iot/raspberry-pi/projects/technotiki" class="button button--small">Check Out The Knowledge Guide</a>

Here are some guides I used to help come up with our end product:
- <a href="https://www.circuits.dk/temperature-logger-running-on-raspberry-pi/">https://www.circuits.dk/temperature-logger-running-on-raspberry-pi/</a>
- <a href="https://www.definit.co.uk/2018/07/monitoring-temperature-and-humidity-with-a-raspberry-pi-3-dht22-sensor-influxdb-and-grafana/">https://www.definit.co.uk/2018/07/monitoring-temperature-and-humidity-with-a-raspberry-pi-3-dht22-sensor-influxdb-and-grafana/</a>
- <a href="https://www.terminalbytes.com/temperature-using-raspberry-pi-grafana/">https://www.terminalbytes.com/temperature-using-raspberry-pi-grafana/</a>
- <a href="http://www.hietala.org/temperature-reading-influxdb.html">http://www.hietala.org/temperature-reading-influxdb.html</a>
- <a href="https://ainsey11.com/2019/04/07/raspberry-pi-temperature-sensor-that-logs-to-grafana/">https://ainsey11.com/2019/04/07/raspberry-pi-temperature-sensor-that-logs-to-grafana/</a>
- <a href="https://pimylifeup.com/raspberry-pi-influxdb/">https://pimylifeup.com/raspberry-pi-influxdb/</a>
- <a href="https://grafana.com/grafana/download?platform=arm">https://grafana.com/grafana/download?platform=arm</a>
- <a href="https://grafana.com/grafana/plugins/farski-blendstat-panel/installation">https://grafana.com/grafana/plugins/farski-blendstat-panel/installation</a>
- <a href="https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-with-ssl-as-a-reverse-proxy-for-jenkins">https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-with-ssl-as-a-reverse-proxy-for-jenkins</a>