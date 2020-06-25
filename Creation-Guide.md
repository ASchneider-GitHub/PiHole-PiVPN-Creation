
## Guide Steps

1. [Burning Raspberry Pi OS Lite](#Burning-Raspberry-Pi-OS-Lite)
2. [Enabling SSH and getting the RasPi IP address](#Enabling-SSH-and-getting-the-RasPi-IP-address)
3. [Connecting-to-the-RasPi-and-updating-it](#Updating-the-RasPi-and-installing-QoL-packages)
4. [Installing and configuring PiHole](#Installing-and-configuring-PiHole)
5. [Accessing the PiHole and adding blocklists/regex filters](#Accessing-the-PiHole-and-adding-blocklistsregex-filters)
6. [Creating and configuring whitelists](#Creating-and-configuring-whitelists)
7. [Updating the Gravity using SSH](#Updating-the-Gravity-using-SSH)
8. [Deploying the PiHole over the network](#Deploying-the-PiHole-over-the-network)
---
### Burning Raspberry Pi OS Lite
1. Download a copy of "Raspberry Pi OS (32-bit) Lite" from [https://raspberrypi.org](https://www.raspberrypi.org/downloads/raspberry-pi-os/).
2. Install your choice of ISO burner. I recommend [balenaEtcher](https://www.balena.io/etcher), but other options such as [UNetbootin](https://unetbootin.github.io/) and [Rufus](https://rufus.ie/) will also work.
3. Burn your downloaded copy of "Raspberry Pi OS (32-bit) Lite" to your micro SD card. Once it's complete, remove and remount your micro SD card so your computer can read the new partition layout.
---
### Enabling SSH and getting the RasPi IP address
1. After remounting your micro SD card, open up volume called `boot`.
2. Create a blank file called `ssh` inside of the root of the `boot` drive. Make sure there is no extension attached to this file. If you're unsure of how to do this, you can download one [here](https://raw.githubusercontent.com/ASchneider-GitHub/PiHole-Creation/master/ssh) (right click `here` and select "Save As"). Make sure the filename is not changed from `ssh`, and no extension was added.
3. Once the file is in the root of the `boot` volume, unmount the micro SD card from your computer, insert it into the RasPi, and power it on. You'll also need to make sure it has a working Ethernet connection. For a RasPi Zero, the right-hand Micro-USB port is power, and the left-hand Micro-USB port supports an Ethernet connection with an [RJ45->Micro-USB adapter](https://www.amazon.com/dp/B00RM3KXAU/)
4. Once the RasPi has powered on, you'll need to find the IP address that it has grabbed from your network's DHCP server. The easiest way to do this is with a network scanner. I recommend [Angry IP Scanner](https://angryip.org/) but many others will work. Once you've installed a scanner, run it and find the device with the hostname "raspberrypi" or "raspberrypi.local". Make note of the IP address that it's currently using, as well as it's MAC address.
5. This next step will vary wildly depending on which router you use in your house, but you need to log into whatever router acts as your DHCP server and create a static IP address reservation for the RasPi. Most routers will let you enter the MAC address from the previous step, enter an IP you'd like to reserve for it (remember what you choose), and a name for the reservation.
6. Once the IP address reservation has been made, power off the RasPi and then power it back on so it can grab the newly reserved IP address.
---
### Connecting to the RasPi and updating it
1. Open up a Command Prompt/Terminal session and type `ssh -l pi [reservedIPAddress]`. When it asks for a password, enter `raspberry`. This is the default password.
2. After connecting to the RasPi, type `sudo apt-get update && sudo apt-get upgrade -y` to update all of the currently installed packages
3. Install a text editor of your choice. I prefer vim (`sudo apt-get install vim`), but any other will do. If you opt for something else, steps below may vary slightly.
4. This step is optional, but you most likely want to change the password for your `pi` profile. Do this by typing `passwd`, providing the default password of `raspberry`, and then entering your new password.
---
### Installing and configuring PiHole
1. While still using SSH to access the RasPi, run `sudo curl -sSL https://install.pi-hole.net` | `bash` to begin the automatic installation process. (Include everything from `sudo` to `bash` in the same command. Markdown doesn't handle pipe characters well with inline code)
2. Once the installer appears, the following settings should be used:
	1. 
---
### Accessing the PiHole and adding blocklists/regex filters

---
### Creating and configuring whitelists

---
### Updating the Gravity using SSH

---
### Deploying the PiHole over the network

