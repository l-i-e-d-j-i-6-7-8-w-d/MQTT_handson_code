# An hands-on introduction to MQTT 

This lab aims to offer you an hands-on experience with MQTT. You will perform experiments that will allow you to learn how to "publish" and "subscribe" to data. To this end you will use:
1. your own broker
1. a "sandbox" external broker
1. the **ThingSpeak** and **Ubidots** platform

You will learn how to:
* install and configure an MQTT broker
* interchange data using MQTT clients based on Python and MicroPython for the LoPy
* use MQTT to feed data to cloud based IoT platforms

## Hardware

> All devices in the lab must share the same WLAN.

Each group will use a computer and a LoPy connected via USB through either an extension board or a PySense board. 


# Block 0: Installing the MQTT broker on your computer

For our experiments we will use [**Mosquitto**](https://mosquitto.org/), which is part of the [Eclipse Foundation](http://www.eclipse.org/) and is an [iot.eclipse.org](https://projects.eclipse.org/projects/technology.mosquitto) project. The manual page can be found here [`man page`](https://mosquitto.org/man/mosquitto-8.html).

Detailed installation indications can be found here: https://mosquitto.org/download/ 
As a quick guide:

* Linux distros like Debian/UBUNTU/Raspian already have Mosquitto in their repositories... so it's enough with:
```shell=bash
sudo apt-get update
sudo apt-get install mosquitto mosquitto-clients
```

* with *Ubuntu MATE* maybe you'll need to add this before:
`sudo apt-add-repository ppa:mosquitto-dev/mosquitto-ppa` 

* with Macs, Mosquitto can be installed from the homebrew project. See  http://brew.sh/ and then use “brew install mosquitto”


### Managing the broker


#### ... with Debian/UBUNTU/Raspian
To start and stop its execution use:
```shell=bash
sudo /etc/init.d/mosquitto start/stop
```
if necessary, to avoid that it restarts automatically, do: `sudo stop mosquitto`

To run the broker execute:
```shell=bash
sudo mosquitto –v
```
> note: "-v" stands for "verbose mode" and can be useful at the beginning to see what is going on in the broker. Can be convenient to use a dedicated terminal for the broker to execute in, if the "-v" option is used.

To check if the broker is running you can use the command:
```shell=bash
sudo netstat -tanlp | grep 1883
```
> note: "-tanlp" stands for: tcp, all, numeric, listening, program

alternatively use:
```shell=bash
ps -ef | grep mosquitto
```


#### ... with Mac OS
To start and stop its execution use:
```shell=bash
/usr/local/sbin/mosquitto -v
```
> note: "-v" stands for "verbose mode" and can be useful at the beginning to see what is going on in the broker. Can be convenient to use a dedicated terminal for the broker to execute in, if the "-v" option is used.

or:
```shell=bash
/usr/local/sbin/mosquitto -c /usr/local/etc/mosquitto/mosquitto.conf
```
or:

```shell=bash
brew services start/stop mosquitto
```

To check if the broker is running you can use the command:
```shell=bash
sudo lsof -i -n -P | grep 1883
```

or:
```shell=bash
ps -ef | grep mosquitto
```



## Clients for testing
The broker comes with a couple of useful commands to quickly publish and subscribe to some topic. Their basic syntax is the following. 
```shell
mosquitto_sub -h HOSTNAME -t TOPIC
mosquitto_pub -h HOSTNAME -t TOPIC -m MSG
```
More information can be found:
* https://mosquitto.org/man/mosquitto_sub-1.html
* https://mosquitto.org/man/mosquitto_pub-1.html

---
---

# Block 1: some basic example.

## Set-up
Open three terminals (e.g., `xterm`) in your computer, more or less like this:
![](https://i.imgur.com/KOcNjwz.jpg=400x400)
The biggest terminal on the right will be used to see the execution of the broker, the two smaller terminals will be used to execute the publisher and the subscriber, respectively.

Now, run the broker with the `-v` flag in the biggest terminal.

## Exercises

Let's start with a easy one. In one of the small terminals write:
```shell
mosquitto_sub -t i/LOVE/Python
```
the broker terminal should show something like:

![](https://i.imgur.com/5nMOywi.png)

the broker registered the subscription request of the new client. Now in the other small terminal, execute:
```shell
mosquitto_pub -t i/LOVE/Python -m "Very well."
```
in the broker terminal, after the new registration messages, you'll also see something like:

![](https://i.imgur.com/s7zROiH.png)

meaning that the broker received the published message and that it forwarded it to the subscribed client. In the terminal where `mosquitto_sub` is executing you'll see the actual message appear.

Try now: 
```shell
mosquitto_pub -t i/love/python -m "Not so well"
```
**What happened? Are topics case-sensitive?**

Another useful option of `mosquitto_pub` is `-l`. Execute the following command:
```shell
mosquitto_pub -t i/LOVE/Python -l
```
and start typing some line of text. It sends messages read from stdin, splitting separate lines into separate messages. Note that blank lines won't be sent. You basically obtained a MQTT based **"unidirectional chat"** channel... 

### ... about Keepalive
By the way, if you kept the broker running with the `-v` option until now in a separate window, you can see various lines like:
```
1524673958: Sending PINGRESP to mosqpub|3592-iMac-de-Pi
1524673985: Received PINGREQ from mosqsub|3587-iMac-de-Pi
```
this simply shows that the broker and the client are interchanging these special messages to know whether they are still alive.


### QoS (Quality of Service):
Adding the `-q` option, for example to the `mosquitto_pub` you'll see the extra message that are now interchanged with the broker. For example, doing:
```shell
mosquitto_pub -t i/LOVE/Python -q 2 -m testing
```

you'll get:

![](https://i.imgur.com/wLqMrev.png)

compare this sequence of messages with the one obtained with `-q 0` or with `-q 1`.

### Retained messages:
Normally if a publisher publishes a message to a topic, and *no one is subscribed* to that topic the message is simply discarded by the broker. If you want your broker to remember the last published message, you'll have to use the ```retain``` option. Only one message is retained per topic. The next message published on that topic replaces the retained message for that topic. 
> To set the retain message flag you have to add `-r` using the Mosquitto clients.

So try the following cases, but  **remember now to always execute, for each test, the subscriber after** the publisher:
1. Publish a message with the retain message flag not set, like we did before. What happens?
1. Publish a message with the retain message flag set (`-r`). What happens?
1. Publish several (different) messages with the retain message flag set before starting the subscriber. What happens?
2. Publish a message with the retain message flag **not** set again. What happens?

Finally, how do I remove or delete a retained message? You have to publish a blank message(`-m ""`) with the retain flag set to true which clears the retained message. Try it.

### Public brokers
There are also various public brokers in Internet, also called `sandboxes`. For example:
* `test.mosquitto.org`
    * more infos at: http://test.mosquitto.org/
* `iot.eclipse.org`
    * more infos at: https://iot.eclipse.org/getting-started#sandboxes
* `broker.hivemq.com`
    * more infos at: http://www.hivemq.com/try-out/
        * http://www.mqtt-dashboard.com/
        
we will always access them through port `1883`. 

**Repeat some of the exercise above with one of  these sandboxes (remember to use the `-h` option). Any difference?**





# Block 2: MQTT clients with MicroPython and the LoPy

> **All the code that you will be using is available here https://github.com/pmanzoni/MQTT_handson_code**


## First, some basic code

### The `ufun.py` library
To ease the programming of the following exercises some generic code is provided in a library called ```ufun.py```  available in the repository indicated above. The library provides the code to: 
* `connect_to_wifi()`: connects the LoPy to a WiFi LAN. By properly passing the values `wifi_ssid` and `wifi_passwd`  this function will try three times to connect to the specified AP, exiting if the operation is not possible. 
* `random_in_range()`: generates random numbers in a range, and 
* `set_led_to()` and `flash_led_to()`: simplify the control of the LED. 

_Take a quick look at the code  to understand how it works._


### Installing the MQTT client library in the LoPy

The LoPy devices require a MQTT library to write the client application. The code is available in the [above described repository.](#Block-2-MQTT-clients-with-MicroPython-and-the-LoPy) You basically need to download the **`mqtt.py`** file and copy it in the directory of your project. 


## Let's start: a simple subscriber

The code below represent a simple subscriber. As a first step it connects to the WiFi network available in the lab.

> **Remember to properly assign a value to variables: `wifi_ssid`, `wifi_passwd`, and to personalize the value of `dev_id`.** `dev_id` allows to identify your specific device

> In this case we use the broker `test.mosquitto.org` but you can use any other accessible broker.

```python=
# file: a_simple_sub.py

from mqtt import MQTTClient
import time
import sys
import pycom

import ufun

wifi_ssid = 'LOCAL_AP'
wifi_passwd = ''
dev_id = 'PMtest'

broker_addr = 'test.mosquitto.org'

def settimeout(duration):
   pass

def on_message(topic, msg):
    print("Received msg: ", str(msg), "with topic: ", str(topic))

### if __name__ == "__main__":

ufun.connect_to_wifi(wifi_ssid, wifi_passwd)

client = MQTTClient(dev_id, broker_addr, 1883)
client.set_callback(on_message)

print ("Connecting to broker: " + broker_addr)
try:
	client.connect()
except OSError:
	print ("Cannot connect to broker: " + broker_addr)
	sys.exit()	
print ("Connected to broker: " + broker_addr)

client.subscribe('lopy/lights')

print('Waiting messages...')
while 1:
    client.check_msg()

```

**Now, in a terminal and using `mosquitto_pub`, write the proper command to send some message to the LoPy.**

## A simple publisher

Let's produce some random data using the code below. As before, it first connects to the WiFi network available in the lab.
> **Remember to properly assign a value to variables: `wifi_ssid`, `wifi_passwd`, and `dev_id`.**

> In this case we use the broker `test.mosquitto.org` but you can use any other accessible broker.

```python=
# file: a_simple_pub.py

from mqtt import MQTTClient
import pycom
import sys
import time

import ufun

wifi_ssid = 'LOCAL_AP'
wifi_passwd = ''
dev_id = 'PMtest'

broker_addr = 'test.mosquitto.org'

def settimeout(duration):
   pass

def get_data_from_sensor(sensor_id="RAND"):
    if sensor_id == "RAND":
        return ufun.random_in_range()

### if __name__ == "__main__":

ufun.connect_to_wifi(wifi_ssid, wifi_passwd)

client = MQTTClient(dev_id, broker_addr, 1883)

print ("Connecting to broker: " + broker_addr)
try:
	client.connect()
except OSError:
	print ("Cannot connect to broker: " + broker_addr)
	sys.exit()	
print ("Connected to broker: " + broker_addr)

print('Sending messages...')
while True:
    # creating the data
    the_data = get_data_from_sensor()
    # publishing the data
    client.publish(dev_id+'/sdata', str(the_data))
    time.sleep(1)

```

**Now, in a terminal and using `mosquitto_sub`, write the proper command to read the generated data.**


---

# Block 3: Final exercises
Now let's work on some final exercises to put together most of what we saw in this lab session. Since you'll have to write some MQTT Python program (_not MicroPython_  :smirk:)  for your computer, you have to install the [Paho library](https://www.eclipse.org/paho/clients/python/) that I described in class; it's just one step, execute:

```shell
sudo pip install paho-mqtt
```

## The first
Let's control remotely the color of the LoPy's LED using MQTT.

![block 3, first exercise](https://i.imgur.com/Y7TDE2U.png)


**Code “p1”.** This code runs in the LoPy, so you must use MicroPython, and has to:
* Connect each LoPy to its own 'private' broker; 'private' means that each user should use a different broker, basically the one you installed at the beginning of this Lab session.
* Have the LoPy to change the color of its LED according to the "instructions" it receives using MQTT. Use the functions in library `ufun.py` to control the LED.

**Code “p2”.** This code runs in a computer, so you must use Python, and has to:
* Connect to the LoPy 'private' broker. 
* Publish the "instructions", using MQTT, to inform the LoPy to which color has to set its LED:
    1) Try first simply using: `mosquitto_pub`
    2) Then, write a program that reads 2 parameters: the broker address and the LED color you want that specific LoPy to show. 
    3) Finally, try to control the LoPy of another group.

## The second
Now repeat the previous exercise but using a unique ("common") broker for the whole lab. It could either be one running in a computer in the lab or a remote one (e.g., test.mosquitto.org). How will you identify a specific LoPy now? 

![block 3, second exercise](https://i.imgur.com/V2q18hb.jpg)


---

# Block 4: Sending data to a cloud based platform

In this block you will experiment about how MQTT can be used to send data to a cloud based platform. This procedure allows you to store your data in a cloud based repository and to analyze your data with software tools made available by the used platform.

## Using ThingSpeak

ThingSpeak is an IoT analytics platform service that allows you to aggregate, visualize and analyze live data streams in the cloud. ThingSpeak provides instant visualizations of data posted by your devices to ThingSpeak. With the ability to execute MATLAB® code in ThingSpeak you can perform online analysis and processing of the data as it comes in. 

### Creating a *channel*
You first have to sign in. Go to https://thingspeak.com/users/sign_up and create your own account. Then you can create your first channel. Like for example:

![](https://i.imgur.com/siPq11m.png =300x400)


When a channel is created it is set as _private_. Set it to **public** as indicated in the figure below:

![](https://i.imgur.com/Dforjus.png)

Now you can take a look at it, as shown in the figure below. No data is plotted for the moment.

![](https://i.imgur.com/CYPVnr4.png)


You need the data in the API Keys section to connect to your channel. 

![](https://i.imgur.com/YZt82yB.png)


### Exercise
ThingSpeak offers either a REST and a MQTT API to work with channels. See here: https://es.mathworks.com/help/thingspeak/channels-and-charts-api.html

For this exercise you will need the documentation specific to **publish a message to update a single channel field** using MQTT. It is here: https://es.mathworks.com/help/thingspeak/publishtoachannelfieldfeed.html


#### First step: use `mosquitto_pub`

Use `mosquitto_pub` to send a value to your channel:

Consider that:
1. the hostname of the ThinSpeak MQTT service is "mqtt.thingspeak.com"
2. the topic you have to use is `channels/<channelID>/publish/fields/field<fieldnumber>/<apikey>` where you have to replace:
    * <channelID> with the channel ID,
    * <fieldnumber> with field number that you want to update, and 
    * <apikey> with the write API key of the channel. 
3. finally, remember that ThingSpeak requires you to:
    * set the PUBLISH messages to a QoS value of 0.
    * set the connection RETAIN flag to 0 (False).
    * set the connection CleanSession flag to 1 (True).
* Be careful!!!:
    * remember to add the string 'fields'
 channels/<channelID>/publish/**fields**/field<fieldnumber>/<apikey>
    * `field<fieldnumber>` means, for example, **field1**
    * use the **Write** API Key

#### Second step: use a Python program.

Using as a reference the code in file `paho-code/example4.py` in the GitHub repository, create a periodic publisher that sends the generated number to your ThingSpeak channel.

#### Third step: use a MicroPython program.

Using as a reference the code of the previous step, create a MicroPython periodic publisher that sends the generated number from your LoPy to your ThingSpeak channel.


## Using Ubidots

Repeat the previous exercise with the Ubidots platform. You will have to first create your free account here: https://app.ubidots.com/accounts/signup/ Then create a device:

![](https://i.imgur.com/CkHHqh3.png)

and then create a "Default" type variable:

![](https://i.imgur.com/ITZeABD.png)

Now we will send data to our device using MQTT. Take a look first to the MQTT API Reference: https://ubidots.com/docs/api/mqtt.html

#### First step: use `mosquitto_pub`

Use `mosquitto_pub` to send a value to your device:

Consider that:
1. the hostname for educational users is "things.ubidots.com". To interact with it, you will need a TOKEN. The easiest way to get yours is clicking on “API Credentials” under your profile tab:

![](https://i.imgur.com/QMXvJL0.png)

In my case I have:

![](https://i.imgur.com/72pXlm0.png)

To connect to the MQTT broker you'll have to use your Ubidots TOKEN as the MQTT username, and leave the password field empty.

2. the topic you have to use is **`/v1.6/devices/{LABEL_DEVICE}/{LABEL_VARIABLE}`** where you have to replace the fields `{LABEL_DEVICE}` (e.g., VLCtesting) and `{LABEL_VARIABLE}`  (e.g., my_value).

4. The data must be represented using JSON. The simplest format is: `{"value":10}` 

So, summing up, to send value 25 to variable `my_value` of device VLCtesting 

```
mosquitto_pub -h things.ubidots.com -u A1E-2DvBg......TsjaOcG4SRuTkgH -P '' 
              -t /v1.6/devices/vlctesting/my_value -m '{"value":25}'
```

> Be careful on the use of `'`and `"` and on the actual identifiers of the device and the variable (e.g., uppercase, lowercase, ...)
![](https://i.imgur.com/EEPGJaR.png =200x200)


You'll get:
![](https://i.imgur.com/xNFjzBv.png =400x300)

So try to repeat all the previous steps with your own device and variable.

#### Second step: use a Python program.

As before, and using as a reference the code in file `paho-code/example5-prod.py` in the GitHub repository, create a periodic publisher that sends the generated number to your Ubidots device.

#### Third step: use a MicroPython program.

Using as a reference the code of the previous step, create a MicroPython periodic publisher that sends the generated number from your LoPy to your Ubidots device.

