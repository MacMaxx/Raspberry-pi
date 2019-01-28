## 1905 RHAV

Repository for documenting Raspberry experiments

# References:
* www.raspberrypi.org
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

# Login
Using `terminal` to `ssh` into raspberry:
```
> ssh pi@<IP address>
```

```
> sudo apt-get update
> sudo apt-get upgrade
```

# Setup

## Packages installed:
* [package name](link to package goes here)
