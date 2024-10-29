# Raspberry PI Journal

Idea: Learn about self hosting, networking and creating relatively safe RPI that's exposed to the Internet. 123

This RPI can and probably will be used as a home server and/or a server that can host multiple web API's (with the intention that they will strictly communicate with one or two devices outside home network).

*note: At the time of writing this md file, you can consider me a beginner to RPI, and thus some of notes might be over explained while others might not be.

**Prerequisites**
- Raspberry PI 4 - further known as RPI
- RPI heatsink (for now nothing more's needed)
- RPI Power Cable
- MicroSD card (size used 32GB)
- MicroSD card reader (optional)
- Ethernet cable (optional)
- RPI case (optional)

## RPI OS Installation
Whole step by step process, with detailed documentation can be found below on [official site](https://www.raspberrypi.com/documentation/computers/getting-started.html).<br>
Shorter version is here (this process assumes that W10 is an OS from which RPI Os is being prepared):

### Install using Imager
1. Download the latest Imager [software](https://www.raspberrypi.com/software/)
   - Choose Device: RPI 4
   - Operating System: Raspberry Pi OS (64-bit) * but since this RPI is being accessed only through SSH, this could've been "lite" version of OS
   - Storage (SD card)
2. Customize OS
   - a username and password
   - Wi-Fi credentials (optional)
   - the device hostname 
   - the time zone
   - your keyboard layout (optional)
   - **remote connectivity** (since we're only connecting via SSH this is a must).
     - Services (second tab) provides option to **enable SSH** and to choose authentication method (password authentication).
   - Lastly Save changes and wait for installation to finish
3. First RPI boot
   - Make sure that the RPI's power cable is unplugged
   - Insert SD card
   - Plug in power cable
   - If RPI does not boot within 2-3 minutes, check the status LED. If it’s flashing, see the LED warning flash codes for [more information](https://www.raspberrypi.com/documentation/computers/configuration.html#led-warning-flash-codes).

You should be good to go from here, as far as I remember.

## Docker containers

Next chapter is dedicated to docker containers I installed on RPI.<br>
The following list is in the order I added them.

- [x] [DuckDNS](https://www.duckdns.org/)
- [x] [WireGuard](https://www.wireguard.com/)
- [ ] NgRok?

### Prerequisites

- Docker
- Docker Compose

---
### DuckDNS Container
source: [Linux Server](https://docs.linuxserver.io/images/docker-duckdns/)

**What is:** A simple but effective DDNS (dynamic DNS) service.

The reason for why I chose DuckDNS, is simple. It is really simple to setup up and to keep up to date with the changes my ISP makes to my public IP address.<br>
Also I'm a cheap a$$ so I didn't want to pay for static IP address (this might change in the future).
Another reason, a colleague of mine recommended it to me, another option is [NoIP](https://www.noip.com/) but this requires a monthly check-in to keep using their services.

Login to duckDNS however you wish and create a subdomain, for a free account the limit is 5 subdomains which should be more than enough for recreational purposes.<br>
Once you create an account, you'll have the access to the token you'll need later on.

The docker container setup is fairly simple:
1. Create a directory (name: `duckdns`) on your RPI (optional, but recommended)
2. Create a file in that directory `docker-compose.yml`
3. Go to the DuckDNS docker image on LinuxServer and copy the docker-compose configuration
4. Environment Changes
   1. Change the value for `TOKEN` (insert your own)
   2. Change the value for `SUBDOMAINS` (comma separated list)
5. Volume changes
   1. Locate them wherever you wish (recommended inside the folder that holds the yml file in the folder named (`config`))
6. Voila you have a docker container that will take care of updating your public IP address for the desired subdomain
7. The only thing you'll need is to start it up with `docker-compose up -d`

By the default if you don't provide the value for `UPDATE_IP` inside the yml file, this container will use duckDns to check your public IP address.<br>
If you want other 3rd party to do that instead (*CloudFlare whoamI* service), you can change the value for `UPDATE_IP` to any of the following values `ipv4`, `ipv6` or `both`. <br>
I **don't** recommend that.


You can also specify that you want to update IPv6, but I didn't need it so good luck with that.

The docker container will try to update your public IP address every 5 minutes + random noise.

---
### WireGuard Container
source: [Linux Server](https://docs.linuxserver.io/images/docker-wireguard/)

**What is:** "A simple yet fast and modern VPN that utilizes state-of-the-art cryptography."<br>
<sub>(you might see the pattern here)</sub>

One of the first thoughts I had, when I bought the RPI is that I want to access my computer station from my laptop wherever I am.<br>
And the answer to my thought is VPN, but I wanted to make my own VPN, so after some research and bugging my colleague (yes, the same one), I decided to go with WireGuard.<br>
Another option was [OpenVPN](https://openvpn.net/), but everywhere I looked, the only thing I found was that it's a lot jankier to setup/maintain than it is WireGuard.
Also since my networking experience was, well, close to nonexistent I opted out for WireGuard.

The docker container setup is fairly simple:
1. Create a directory (name: *wireguard*) on your RPI (optional, but recommended)
2. Create a file in that directory `docker-compose.yml`
3. Go to the WireGuard docker image on LinuxServer and copy the docker-compose configuration
4. Environment Changes
   1. Change the value for `SERVERURL` (subdomain.duckdns.org)
   2. Change the value for `SERVERPORT` (optional, if left blank it will use the default value)
   3. Change the value for `PEERS` (how many, devices you want to be able to connect to the VPN, use integer)
5. Volume changes
   1. Locate them wherever you wish (recommended inside the folder that holds the yml file in the folder named (`config`))
   2. Do the same for modules (recommended inside the folder that holds the yml file in the folder named (`modules`))
6. Ports Changes (optional) if you used any other port for `SERVERPORT`
   1. Map device port to the docker container port (e.g. `- 1234:51820/udp`)
   2. **IMPORTANT** If you want to have the SSH access while being connected to the VPN you must map the ports (e.g. `- 6666:22/tcp`)
7. Voila you have a docker container that will take care of updating your public IP address for the desired subdomain
8. The only thing you'll need is to start it up with `docker-compose up -d`


SSH-ing into RPI while being connected on RPI's network through VPN requires a port to be added e.g.
`ssh username@hostname -p <port>`
e.g. `ssh username@hostname -p 6666`

## Other useful software

### Docker
source: [Docker Engine](https://docs.docker.com/engine/install/ubuntu/)

What is: "Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly." [read more](https://docs.docker.com/get-started/docker-overview/)

How to install:
`curl -sSL https://get.docker.com | sh`

Remove the need to add sudo before executing docker commands:
`sudo usermod -aG docker $USER`

Note: If you installed Docker using the convenience script, you should upgrade Docker using your package manager directly.

---
### Docker Compose
source: docker-compose

What is: Define and run multi-container applications with Docker.

How to install:
`sudo apt install docker-compose`

Note:/

---
### HSTR
I'm using PowerShell and I'm quite fond of their [Predictive IntelliSense](https://learn.microsoft.com/en-us/powershell/scripting/learn/shell/using-predictors?view=powershell-7.4) feature, and the closest I could get to it, at the moment, was "HSTR".<br>
[HSTR](https://github.com/dvorka/hstr) (HiSToRy) is a command line utility that brings improved bash / zsh command completion from the history.
I also bound HSTR to "CTRL + R" instead of reverse in '.bashrc' file.<br>
<sub>Don't forget to source `~/.bashrc` to be able to to use new bind</sub>

---
### Certbot 
TODO:

---
### Let's Encrypt
TODO:

---
### Fail2Ban
TODO:

---
## Network modifications
I wont go into specifics here, because the location for the setup may vary for each router...
### Reserving IP address
I reserved a local IP address for RPI so that SSH-ing into it wont cause any issues (hah).<br>
That allows me to give it a host name in local DNS records.<br>
Which further allows me to `ssh username@hostname` instead of `ssh username@ip_address`.
<sub>(note: hostname requires a dot "." in it so that it will be fully qualified.)
[read more_1](https://superuser.com/questions/1078467/why-cannot-i-ping-computer-name-without-dot/1078468);
[read more_2](https://superuser.com/questions/1078460/why-do-i-need-to-add-a-period-after-hostname-in-order-to-get-dns-resolution-to-w)</sub>

### Port Forwarding (Wireguard, etc.)
Added a port forwarding rule for Wireguard that's running on RPI, so that all the traffic that hit's my router on specific port is redirected to RPI, specifically docker container running Wireguard.<br>
Added this port forwarding for VPN.
Probably another port will be forwarded but that will happen in the future.

## Other additions/changes/modifications
Personally, I dislike motd and that's the first thing when I SSH'd into RPI, I removed it by deleting the content of **motd** file<br>
`sudo nano /etc/motd`.

---
Additionally I added a banner that shows up before someone tries to access RPI through SSH ([link](https://www.tecmint.com/ssh-warning-banner-linux/))
Steps to create a banner for SSH
1. Position in /etc/ssh/ folder and write your banner
2. Create a 'sshd-banner' file inside that folder
3. Locate '#Banner none' directive inside ssh_config
4. Replace 'none' with /etc/ssh/sshd-banner and remove comment
5. Restart sshd service and you're done, and on next login, before being prompted for password, you'll see the banner

```bash
cd /etc/ssh/
sudo nano sshd-banner
sudo nano sshd_config
sudo systemctl restart sshd
```

---
Also I removed the message about kernel version after login.<br>
Inside '10-uname' file I removed both shebang and 'uname -snrvm'. (who knows I might return it at some point)
```bash
cd /etc/update-motd.d/
sudo nano 10-uname
```

## Notes:

## TODO: 
Once this gets too messy, create a hierarchy for files