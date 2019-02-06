## 1905 RHAV

Repository for documenting Raspberry Pi experiments

![Raspberry Pi 3B](https://github.com/MacMaxx/Raspberry-pi/blob/master/RaspberryPi3.jpg "Image of the Pi")
      
# References:
* www.raspberrypi.org
* [w3schools](https://www.w3schools.com/nodejs/nodejs_raspberrypi.asp)
* [Markdown cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)

# Initial setup
1. Setup SD card with raspbian as described on https://www.raspberrypi.org/downloads/raspbian/ and follow [installation guide](https://www.raspberrypi.org/documentation/installation/installing-images/README.md)

>BEWARE: To enable `ssh` for a headless clean install, place file named ssh in boot partition on SD card as described [here](https://www.raspberrypi.org/documentation/remote-access/ssh/README.md)

2. Wire connect Raspberry to network (en0)

3. To figure out IP address of Raspberry Pi in network use terminal and type:
```
shell> arp -an | grep b8:27:eb
```
> Mac address of Pi always starts with b8:27:eb

4. Install avahi-deamon as described [here](https://www.howtogeek.com/167190/how-and-why-to-assign-the-.local-domain-to-your-raspberry-pi/)
```
shell> sudo apt-get install avahi-daemon
```
This will expose the Raspberry Pi as `hostname.local` (by default; raspberrypi.local)

5. Set hostname
```
shell> sudo raspi-config (I set hostname to `raspberry`)
```

# Login
Using `terminal` to `ssh` into raspberry:
```
shell> ssh pi@raspberry.local
```
Password: default password since I'm running pi in local network only.

```
shell> sudo apt update         //update installed packages
shell> sudo apt upgrade        //upgrade installed packages
shell> sudo apt dist-upgrade   //upgrade installed distribution (raspbian)
```

> BEWARE: in case of repeating `The authenticity of host 'raspberrypi.local <snip>' can't be established.
ECDSA key fingerprint is <snip>.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'raspberrypi.local,<snip>' (ECDSA) to the list of known hosts.` warnings (after reinstalling Raspberry), clear the .ssh/known_hosts file on the local machine from which you `ssh` into the Raspberry


# Setup

Search for available packages:
``` 
shell> apt search <packagename>
```

## Packages installed:
### Mosquitto MQTT Broker (&client for testing)

* [Mosquitto MQTT Broker](https://mosquitto.org/)
```
shell> sudo apt install mosquitto
```
> TODO: MAKE MOSQUITTO START @ RASPBERRY BOOT


* [Mosquitto MQTT Client](https://mosquitto.org/)

Required for testing mosquitto installation.

```
shell> sudo apt install mosquitto-clients
```
#### Testing Mosquitto setup
Open two ssh teminals to raspberry; 1 for subscribing, the other for publishing.

```
shell1> mosquitto_sub -d -t mytopic                    //start as deamon and subscribe to mytopic
shell2> mosquitto_pub -d -t mytopic -m "rik is gek" .  //start as deamon and publish to mytopic
```
Should yield something alike:
```
Client mosqsub/9585-raspberry received PUBLISH (d0, q0, r0, m0, 'mytopic', ... (10 bytes))
rik is gek
```
Alternative MQTT client for Mac; [mqttbox](http://workswithweb.com/mqttbox.html)

### MySQL server and client (for maintenance & testing)
General information:
* [MySQL](https://www.mysql.com/)
* [Installation instructions](https://raspberrytips.nl/mysql-installeren-op-raspberry-pi/)

####Istallation:
```
shell> sudo apt install mysql-server
```
For maintenance of DB login as ```root``` in MySQL
```
shell> sudo mysql -p -u root
```
Password = admin
#### Setup:
* Login to mySQL
* Create a new database for storing massages posted in MQTT topics
```
MariaDB [(none)]> CREATE DATABASE mqtt;
```
* Set ```mqtt``` database for default use
```
MariaDB [(none)]> USE mqtt;
```
Add ```mqtt_user``` for retrieving mqtt database data; set password = ```mqtt```
```
MariaDB [mqtt]> GRANT ALL PRIVILEGES ON mqtt.* TO mqtt_user@localhost IDENTIFIED BY 'mqtt';
```
Add ```mqtt``` for pushing mqtt data into mqtt database; set password = ```mqtt```
```
MariaDB [mqtt]> GRANT ALL PRIVILEGES ON mqtt.* TO mqtt@localhost IDENTIFIED BY 'mqtt';
```
