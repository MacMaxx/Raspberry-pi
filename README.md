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
> arp -an | grep b8:27:eb
```
> Mac address of Pi always starts with b8:27:eb

4. Install avahi-deamon as described [here](https://www.howtogeek.com/167190/how-and-why-to-assign-the-.local-domain-to-your-raspberry-pi/)
```
> sudo apt-get install avahi-daemon
```
This will expose the Raspberry Pi as `hostname.local` (by default; raspberrypi.local)

5. Set hostname
```
> sudo raspi-config (I set hostname to `raspberry`)
```

# Login
Using `terminal` to `ssh` into raspberry:
```
> ssh pi@raspberry.local
```

```
> sudo apt update         //update installed packages
> sudo apt upgrade        //upgrade installed packages
> sudo apt dist-upgrade   //upgrade installed distribution (raspbian)
```

> BEWARE: in case of repeating `The authenticity of host 'raspberrypi.local <snip>' can't be established.
ECDSA key fingerprint is <snip>.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'raspberrypi.local,<snip>' (ECDSA) to the list of known hosts.` warnings (after reinstalling Raspberry), clear the .ssh/known_hosts file on the local machine from which you `ssh` into the Raspberry


# Setup

Search for available packages:
``` 
> apt search <packagename>
```

## Packages installed:
* [Mosquitto MQTT Broker](https://mosquitto.org/)
```
> sudo apt install mosquitto
```

* [Mosquitto MQTT Client](https://mosquitto.org/)
Required for testing mosquitto installation.

```
> sudo apt install mosquitto-clients
```
