# IoT on Google Cloud over DTN
This project has been developed by Dr Lara Suzuki :woman_technologist: [![Twitter](https://img.shields.io/twitter/url/https/twitter.com/larasuzuki.svg?style=social&label=Follow%20%40larasuzuki)](https://twitter.com/larasuzuki) and supervised by Vint Cerf :technologist: [![Twitter](https://img.shields.io/twitter/url/https/twitter.com/vgcerf.svg?style=social&label=Follow%20%40vgcerf)](https://twitter.com/vgcerf), both at Google Inc.

[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://GitHub.com/lasuzuki/StrapDown.js/graphs/commit-activity)
[![MIT license](https://img.shields.io/badge/License-MIT-blue.svg)](https://lbesson.mit-license.org/)
![Profile views](https://gpvc.arturio.dev/lasuzuki)
[![GitHub contributors](https://img.shields.io/github/contributors/Naereen/StrapDown.js.svg)](https://GitHub.com/lasuzuki/StrapDown.js/graphs/contributors/)
[![Open Source Love svg1](https://badges.frapsoft.com/os/v1/open-source.svg?v=103)](https://github.com/ellerbrock/open-source-badges/)
[![saythanks](https://img.shields.io/badge/say-thanks-ff69b4.svg)](https://saythanks.io/to/lasuzuki)

[![ForTheBadge built-with-science](http://ForTheBadge.com/images/badges/built-with-science.svg)](https://GitHub.com/lasuzuki/)

In this tutorial we will demonstrate how to connect a Raspberry Pi and Sensor Hat onto Google Cloud using cloud Pub/Sub on `host 1` and serving the messages over DTN to `host 2`. This tutorial follows the [Running DTN on Google Cloud using a Two-Node Ring](https://github.com/lasuzuki/dtn-gcp-2nodes) tutorial and uses the configurations of `host 1` and `host 2` as described in the [rc files](https://github.com/lasuzuki/dtn-gcp-2nodes/tree/main/rc%20files) of that repo.

# Setting up Raspbberry Pi and the Sense Hat
In this tutorial we use **Raspberry Pi 4 model B (2018)** and **Sense Hat Version 1.0**. 

<img src="https://github.com/lasuzuki/dtn-gcp-iot/blob/main/blob/raspberrypi.jpg" width=400 align=center>

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
## SSL Certificate - RSA with X509 wrapper
In order to authenticate in Google Cloud IoT Core, we need a SSL certificate. We will create an RSA with X509 wrapper. For this, execute the following command on your Pi's terminal:

````
$ openssl req -x509 -newkey rsa:2048 -keyout sensing_private.pem -nodes -out demo.pub -subj “/CN=unused”
````

# Setting up Google Cloud Platform
The First step is to enable Cloud IoT on Google Cloud. If you followed the instructions of tutorial [DTN 101 - Running the Interplanetary Internet on Google Cloud](https://github.com/lasuzuki/dtn-gcp), you will already have Cloud IoT enabled. If not, once you open Google Cloud Console, at the search bar type `IoT Core`. Once a the IoT page is loaded, click `Enable` so that the appropriate permissions are granted.

## Setting up your Pi on Google Cloud IoT Core and Pub/Sub
Once IoT Core is enabled, at the top left corner, click on the hamburger icon, scrow down until you find `IoT Core`. A page of Registries will open. The following steps demonstrates how to create a Registry for your Raspberry Pi on Google IoT Core.
1. Click on `Create Registry`
2. Give an id for your registry. In our case we called it `terrestrial-station`
3. Select the region closest to you
4. Under `Cloud Pub/Sub topics`, in the dropdown menu, select `Create a Topic`
5. In the pop-up window type a name for your topic. In our case we gave the topic name `sensing`
6. Back in the dropdown menu, select the topic you have just created
7. Hit `Create`

Back in the registry window you will see that your registry, `terrestrial-station` has been created. To register your Raspberry Pi on the registry, follow the steps below:
1. Click on the `terrestrial-station` registry
2. Under `IoT Core` click `Devices`
3. At the top of the page, click `Create a Device`
4. Give an id to your device. In our case we named it as `sensing-hat`
5. Click on the link `Communication, Cloud Logging, Authentication`
6. Under `Authentication`, select Public Key Format `RS256_X509`
7. To confirm that your Pi is okay to send messages to Google IoT Core, in the `Public Key Value` text box, paste the value of your Pi's public key

To copy the content of your Pi's public key, on the Pi's terminal run:
````
$ cat demo.pub
````

Copy everything, including the tages, between 
````
-----BEGIN PUBLIC KEY-----
-----END PUBLIC KEY-----
````
and paste it in the `Public Key Value` textbox.

Finally, the last security configuration is to obtail the Google roots.pem file so your device knows it’s communicating with Google Cloud IoT Core. On the Pi run
````
$ wget https://pki.google.com/roots.pem
````

## Create a Subscription to listen to the Pub/Sub Topic
On the Google Cloud Console, type Pub/Sub on the search bar, or pick up from the hamburger menu on the top left side of the window. Follow the steps below to create a subscription to listen to the topic `sensing`:
1. Click `Create Subscription`
2. Give an id to your subscription. In our case we named it `listener`
3. On the dropdown menu select the topic you created (in our case: `sensing`)
4. Click `Create`

Now you have all the pieces needed to send telemetry data from your Pi to Google Cloud IoT Core!

# Send telemetry data from Raspberry Pi to Google Cloud

The code on this repository named `sense.py` is based on the implementation of [GabeWeiss](https://github.com/GabeWeiss/GCP_Quick_Starts). 

In the code, edit the following fields:

```python
ssl_private_key_filepath = '/home/pi/sensing_private.pem'
ssl_algorithm = 'RS256'
root_cert_filepath = '/home/pi/roots.pem'
project_id = 'dtn-host-iot-297209'
gcp_location = 'us-central1'
registry_id = 'terrestrial-station'
device_id = 'sensing-hat'
```

Once you have configured the above parameters in the file sense.py, on your Raspberry Pi run the command:
````
$ python3 sense.py
````
# Send telemetry data to from host 1 to host 2 via DTN

Log into the VM `host 1`. In the VM go to the base directory of ION and create a folder named `dtn`
````
$ mkdir dtn
````
CD into dtn directory, and clone the file named `iot.py`. In this file configure the following parameters:
```python
subscription_path = subscriber.subscription_path(
  'ID_OF_YOUR_GOOGLE_CLOUD_PROJECT', 'ID_OF_YOUR_SUBSCRIPTION')
```
And add `host 2` as the receiver of the telemetry data:
```python
os.system(f'echo "{value}" | bpsource ipn:2.1')
```
On the terminal of `host 1` and `host 2`, start ion:
````
$ ionstart -I hostX.rc #where X is the number of the host
````
On the terminal of `host 2`, start `bpsink`
````
$ bpsink ipn:2.1 &
````
On the terminal of `host 1`, start `iot.py`
````
$ python3 iot.py
````
On the terminal of `host 1` you should see the print out of the telemetry data received as below:

<img src="https://github.com/lasuzuki/dtn-gcp-iot/blob/main/blob/host1.png" width=600 align=center>

On the terminal of `host 2` you should see the payloads delivered. Please note that messages beyond 80 characters are not shown on `bpsink`:

<img src="https://github.com/lasuzuki/dtn-gcp-iot/blob/main/blob/host2.png" width=600 align=center>
