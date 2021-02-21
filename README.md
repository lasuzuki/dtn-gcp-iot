# IoT on Google Cloud over DTN

In this tutorial we will demonstrate how to connect a Raspberry Pi and Sensor Hat onto Google Cloud using cloud Pub/Sub on `host 1` and serving the messages over DTN to `host 2`. This tutorial follows the [Running DTN on Google Cloud using a Two-Node Ring](https://github.com/lasuzuki/dtn-gcp-2nodes) tutorial and uses the configurations of `host 1` and `host 2` as described in the [rc files](https://github.com/lasuzuki/dtn-gcp-2nodes/tree/main/rc%20files) of that repo.

# Setting up Raspbberry Pi and the Sense Hat
In this tutorial we use **Raspberry Pi 4 model B (2018)** and **Sense Hat Version 1.0**. 

<img src="https://github.com/lasuzuki/dtn-gcp-3nodes/blob/main/blob/raspberrypi.jpg" width=400 align=center>

The first step is to be sure your Pi can connect to the Internet. You can either plug in an ethernet cable, or if you’re using WiFi, scan for networks your Pi can see. Plug your Pi in a monitor, and when it starts, at the top right corner you can find the wifi logo. Select the network you want to connect to. Once that is connected open your browser to check whether you can access the Internet.

## Library dependency setup
The first thing to do is to make sure that the places where Raspberry Pi will be getting its libraries from is current. For this, on your Pi's terminal, run the following command:
````
$ sudo apt-get update
````
The next step is to securely connect to Google Cloud IoT Core. For this we will use JWT to handle authentication (library `pyjwt`). The meta-model for communication  used on the Google Cloud Pub/Sub is based on publish/subscribe messaging technology provided by the **MQTT (MQ Telemetry Transport) protocol** (library paho-mqtt). MQTT is a topic-based publish/subscribe communications protocol that is designed to be open, simple, lightweight, easy-to-implement, and  efficient in terms of processor, memory, and network resources.   

On your Pi's terminal run the following commands
````
$ sudo apt-get install build-essential
$ sudo apt-get install libssl-dev
$ sudo apt-get install python-dev
$ sudo apt-get install libffi-dev
$ sudo pip install paho-mqtt
````
For encryption, run the install the `pyjwt` library and its dependency, the `cryptography` library .
````
$ sudo pip install pyjwt
$ sudo pip install cryptography
````
For telemetry data we are using Sense Hat. Sense Hat is composed by telemetry sensors such as temperature, accelerometer, humidity and pressure. To install the library for Sense Hat, run the command:
````
$ sudo apt get install sense-hat
````





