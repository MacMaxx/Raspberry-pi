## 1905 RHAV

Repository for documenting Raspberry Pi experiments

![Raspberry Pi 3B](https://github.com/MacMaxx/Raspberry-pi/blob/master/RaspberryPi3.jpg "Image of the Pi")
      
# References:
* www.raspberrypi.org
* [w3schools](https://www.w3schools.com/nodejs/nodejs_raspberrypi.asp)
* [Markdown cheatsheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)
* [Ubiquiti](https://www.ui.com/)
* [Unifi Controller on RaspberryPi](https://raspberrypi.tilburgs.com/unifi-controller/)

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
6. Set "wait for network on boot";
I set "wait for network on boot" to ensure Pi is connected to network before starting all services; else this might e.g. mysql to fail booting.

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
### Unifi Controller

* [Unifi Controller on RaspberryPi](https://raspberrypi.tilburgs.com/unifi-controller/)

Just applied all instructions as suggested, being:

```
shell > sudo apt -y install oracle-java8-jdk
shell > sudo apt -y install dirmngr
shell > echo 'deb http://www.ui.com/downloads/unifi/debian stable ubiquiti' | sudo tee /etc/apt/sources.list.d/100-ubnt-unifi.list
shell > sudo wget -O /etc/apt/trusted.gpg.d/unifi-repo.gpg https://dl.ubnt.com/unifi/unifi-repo.gpg
shell > sudo apt update
shell > sudo apt -y install unifi
shell > sudo systemctl stop mongodb
shell > sudo systemctl disable mongodb
shell > sudo reboot
```

After reboot the Unifi Controller should be reacheable via browseer at http://raspberry.local:8443 or http://<ip of raspberry pi>:8443. Which failed in my case ("This site can't be reached."), so continued with following:
      
```
shell > sudo su
shell > apt install dirmngr
shell > echo deb http://ppa.launchpad.net/openjdk-r/ppa/ubuntu trusty main >> /etc/apt/sources.list.d/glennr-install-script.list
shell > apt-key adv --keyserver keyserver.ubuntu.com --recv EB9B1D8886F44E2A
shell > apt-get update
shell > apt-get install openjdk-8-jre-headless
```
After which to stop & restart the Unifi service:

```
shell > service unifi stop
shell > touch /etc/default/unifi
shell > apt purge oracle-java8-jdk -y
shell > sed -i 's/^JAVA_HOME/#JAVA_HOME/' /etc/default/unifi
shell > echo "JAVA_HOME="$( readlink -f "$( which java )" | sed "s:bin/.*$::" )"" >> /etc/default/unifi
shell > service unifi start
```

Still unifi controller fails to start ('service unifi status' ... failed!')
So resided to [install scripts by UI Glenn](https://community.ui.com/questions/UniFi-Installation-Scripts-or-UniFi-Easy-Update-Script-or-UniFi-Lets-Encrypt-or-Ubuntu-16-04-18-04-/ccbc7530-dd61-40a7-82ec-22b17f027776)

This resolved Unifi failing to start.
webinterface is available at https://<raspberry ip>:8443

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
Create table to contain all the messages received from mqtt
```
mqtt@mysql> CREATE TABLE `messages` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `topic` varchar(255) NOT NULL,
  `message` varchar(255) NOT NULL,
  `timestamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
)
```

#### Node.js & mySQL/Mosquitto modules
Node.js is default installed on Raspbian. While Node Package Manager (npm) isn't:
```
shell> sudo apt install npm
```

```mysql```module requires installation using Node Package Manager (npm) as described on https://www.w3schools.com/nodejs/nodejs_mysql.asp:
```
shell> sudo npm install mysql
```
Create a mysql_test.js file containing:
```
var mysql = require('mysql');

var con = mysql.createConnection({
  host: "localhost",
  user: "mqtt",
  password: "mqtt"
});

con.connect(function(err) {
  if (err) throw err;
  console.log("Connected to mysql-db.");
});
```

Run file using node should yield ```Connected to mysql-db.```;
```
shell> node mysql_test.js
```





SELECT EXISTS (SELECT * FROM information_schema.tables WHERE table_schema = 'mqtt' AND table_name = 'topic');
