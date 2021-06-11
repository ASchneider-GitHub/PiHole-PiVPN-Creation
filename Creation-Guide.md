## Guide Steps

1. [Burning Raspberry Pi OS Lite](#Burning-Raspberry-Pi-OS-Lite)
2. [Enabling SSH and getting the RasPi IP address](#Enabling-SSH-and-getting-the-RasPi-IP-address)
3. [Connecting to the RasPi and updating it](#Connecting-to-the-RasPi-and-updating-it)
4. [Installing and configuring PiHole](#Installing-and-configuring-PiHole)
5. [Accessing the PiHole and adding blocklists/regex filters](#Accessing-the-PiHole-and-adding-blocklistsregex-filters)
6. [Creating and configuring whitelists](#Creating-and-configuring-whitelists)
7. [Deploying the PiHole over the network](#Deploying-the-PiHole-over-the-network)
8. [Miscellaneous Notes](#Miscellaneous-Notes)

---
### Burning Raspberry Pi OS Lite
1. Download a copy of "Raspberry Pi OS (32-bit) Lite" from [https://raspberrypi.org](https://www.raspberrypi.org/downloads/raspberry-pi-os/).
2. Install your choice of ISO burner. I recommend [balenaEtcher](https://www.balena.io/etcher), but other options such as [UNetbootin](https://unetbootin.github.io/) and [Rufus](https://rufus.ie/) will also work.
3. Burn your downloaded copy of "Raspberry Pi OS (32-bit) Lite" to your micro SD card. Once it's complete, remove and remount your micro SD card so your computer can read the new partition layout.

---
### Enabling SSH and getting the RasPi IP address
1. After remounting your micro SD card, open up volume called `boot`.
2. Create a blank file called `ssh` inside of the root of the `boot` drive. Make sure there is no extension attached to this file. If you're unsure of how to do this, you can download one [here](https://raw.githubusercontent.com/ASchneider-GitHub/PiHole-Creation/master/ssh) (right click `here` and select "Save Link As..."). Make sure the filename is not changed from `ssh`, and that no extension was added.
3. Once the file is in the root of the `boot` volume, unmount the micro SD card from your computer, insert it into the RasPi, and power it on. You'll also need to make sure it has a working Ethernet connection. For a RasPi Zero, the right-hand Micro-USB port is power, and the left-hand Micro-USB port supports an Ethernet connection with an [RJ45->Micro-USB adapter](https://www.amazon.com/dp/B00RM3KXAU/)
4. Once the RasPi has powered on, you'll need to find the IP address that it has grabbed from your network's DHCP server. The easiest way to do this is with a network scanner. I recommend [Angry IP Scanner](https://angryip.org/) but many others will work. Once you've installed a scanner, run it and find the device with the hostname "raspberrypi" or "raspberrypi.local". Make note of the IP address that it's currently using, as well as it's MAC address.
5. This next step will vary wildly depending on which router you use in your house, but you need to log into whatever router acts as your DHCP server and create a static IP address reservation for the RasPi. Most routers will let you enter the MAC address from the previous step, an IP you'd like to reserve for it (remember what you choose here), and a name for the reservation.
6. Once the IP address reservation has been made, power off the RasPi and then power it back on so it can grab the newly reserved IP address.

---
### Connecting to the RasPi and updating it
1. Open up a Command Prompt/Terminal session and type `ssh -l pi [reservedIPAddress]`. When it asks for a password, enter `raspberry`. This is the default password.
2. After connecting to the RasPi, type `sudo apt-get update && sudo apt-get upgrade -y` to update all of the currently installed packages
3. Install a text editor of your choice. I prefer vim (`sudo apt-get install vim`), but any other will do. If you opt for something else, steps below may vary slightly.
4. This step is optional, but you most likely want to change the password for your `pi` profile. Do this by typing `passwd`, providing the default password of `raspberry`, and then entering your new password.

---
### Installing and configuring PiHole
1. While still using SSH to access the RasPi, run `sudo curl -sSL https://install.pi-hole.net | bash` to begin the automatic installation process. (Include everything from `sudo` to `bash` in the same command. Markdown doesn't handle pipe characters well with inline code)
2. Once the installer shows the message "This installer will transform your device into a network-wide ad blocker!", the following settings should be used:
	1. **Upstream DNS Provider:** User's choice, but I lean towards Cloudflare. Arrow up or down until you're hovering over your choice, and then press enter.
	2. **Third party block list(s):** I don't use any of these, but if you don't select at least one, the installation breaks. Deselect all but one entry by using the arrow keys and spacebar, and then press enter. The blocklist you choose here will be removed later anyway.
	3. **Select Protocols:** You'll want to leave these both selected. Press enter to continue.
	4. **Static IP Address:** The IP address shown should match the IP address that you've reserved on the router. If it doesn't, cancel the installation with `Esc`, restart the RasPi by unplugging the power source and plugging it back in, then try running the installer again. If it does match, just press enter.
	5. **Installing Web Interface:** Leave it marked as "on" and press enter.
	6. **Installing Web Server:** Leave it marked as "on" and press enter.
	7. **Logging Queries:** Leave it marked as "on" and press enter.
	8. **Select a Privacy Mode:** Make sure "Show Everything" is selected with the arrow keys and spacebar, then press enter.
	9. **Installation Complete!:** If you see this screen, the PiHole was created successfully!
3.  Change the password for the PiHole admin interface using the command `pihole -a -p`. You can press enter to remove the password, or you can choose a new one.

---
### Accessing the PiHole and adding blocklists/regex filters
1. Once you've changed the password for the admin interface, you can now go to `http://[reservedIPAddress]/admin` to access the admin panel for the PiHole.
2. To add blocklists, go to "Group Management" on the left-hand navigation bar, and select "Adlists" from the drop-down.
3. Delete the single entry in the "List of configured adlists" section to completely clear the PiHole's gravity.
4. Copy and paste the entire block below into the "Address" field at the top of the page, and press "Add". This will enter all 12 URLs as individual entries in the Gravity:
	```
    https://hosts.oisd.nl
    https://blocklistproject.github.io/Lists/abuse.txt
    https://blocklistproject.github.io/Lists/ads.txt
    https://blocklistproject.github.io/Lists/crypto.txt
    https://blocklistproject.github.io/Lists/drugs.txt
    https://blocklistproject.github.io/Lists/fraud.txt
    https://blocklistproject.github.io/Lists/gambling.txt
    https://blocklistproject.github.io/Lists/malware.txt
    https://blocklistproject.github.io/Lists/phishing.txt
    https://blocklistproject.github.io/Lists/ransomware.txt
    https://blocklistproject.github.io/Lists/redirect.txt
    https://blocklistproject.github.io/Lists/scam.txt
    https://blocklistproject.github.io/Lists/tracking.txt
	```
5. Once the 12 entries above have been added, switch back to your SSH session with the RasPi, and run the command `pihole -g`. This will download all of the blockslists and add them to the PiHole's Gravity. This step can take a while, as it has to deal with over a million unique domains. 

	***Note:*** If the console shows that a blocklist retrieval failed, you can remove it from the "Adlists" page on the admin panel and replace it with a backup version from below once the initial update is complete. After you have done so, run `pihole -g` again to update it. These lists are manually updated every so often. They may not be 100% up-to-date, but they're a good starting point.
	
	Backups:
	```
	https://raw.githubusercontent.com/ASchneider-GitHub/PiHole-Creation/master/Raw-Lists/OISD-blocklist.txt
	https://raw.githubusercontent.com/ASchneider-GitHub/PiHole-Creation/master/Raw-Lists/BlocklistProject-abuse.txt
	https://raw.githubusercontent.com/ASchneider-GitHub/PiHole-Creation/master/Raw-Lists/BlocklistProject-ads.txt
	https://raw.githubusercontent.com/ASchneider-GitHub/PiHole-Creation/master/Raw-Lists/BlocklistProject-crypto.txt
	https://raw.githubusercontent.com/ASchneider-GitHub/PiHole-Creation/master/Raw-Lists/BlocklistProject-drugs.txt
	https://raw.githubusercontent.com/ASchneider-GitHub/PiHole-Creation/master/Raw-Lists/BlocklistProject-fraud.txt
	https://raw.githubusercontent.com/ASchneider-GitHub/PiHole-Creation/master/Raw-Lists/BlocklistProject-gambling.txt
	https://raw.githubusercontent.com/ASchneider-GitHub/PiHole-Creation/master/Raw-Lists/BlocklistProject-malware.txt
	https://raw.githubusercontent.com/ASchneider-GitHub/PiHole-Creation/master/Raw-Lists/BlocklistProject-phishing.txt
	https://raw.githubusercontent.com/ASchneider-GitHub/PiHole-Creation/master/Raw-Lists/BlocklistProject-ransomware.txt
	https://raw.githubusercontent.com/ASchneider-GitHub/PiHole-Creation/master/Raw-Lists/BlocklistProject-redirect.txt
	https://raw.githubusercontent.com/ASchneider-GitHub/PiHole-Creation/master/Raw-Lists/BlocklistProject-scam.txt
	https://raw.githubusercontent.com/ASchneider-GitHub/PiHole-Creation/master/Raw-Lists/BlocklistProject-tracking.txt
	```
6. Once the blocklists have been added to the "Adlists" menu, click on "Blacklist" in the left-hand navigation bar, and switch to "RegEx filter". Copy and paste the entire block below into the "Regular Expression" field and press "Add to Blacklist". This will create 16 separate RegEx filters for the Gravity to use as a backup ad-catch:
	```
	(.*)\.g00\.(.*)
	^(.+[_.-])?ad([sxv]?[0-9]*|system)[_.-]
	^(.+[_.-])?adse?rv(er?|ice)?s?[0-9]*[_.-]
	^(.+[_.-])?telemetry[_.-]
	^adim(age|g)s?[0-9]*[_.-]
	^adtrack(er|ing)?[0-9]*[_.-]
	^advert(s|is(ing|ements?))?[0-9]*[_.-]
	^aff(iliat(es?|ion))?[_.-]
	^analytics?[_.-]
	^banners?[_.-]
	^beacons?[0-9]*[_.-]
	^count(ers?)?[0-9]*[_.-]
	^mads\.
	^pixels?[-.]
	^stat(s|istics)?[0-9]*[_.-]
	^track(ing)?[0-9]*[_.-]
	```
7. Switch back to your SSH session with the RasPi, and run the command `pihole restartdns reload`. This will add the RegEx entries into the Gravity without re-downloading the entire blocklist database. It saves time, but you're also able to run `pihole -g` if you'd like to do a full update instead.

---
### Creating and configuring whitelists
In order to prevent commonly-used websites from breaking due to false-positives, we can whitelist entire domains from being affected by the PiHole. A convenient tool was created by another individual and is available on GitHub [here](https://github.com/anudeepND/whitelist). We'll use it to update the whitelists en masse.

1. Switch back to your SSH session with the RasPi, and run the commands below, one by one, in the order that they're listed:
```
cd ~
git clone https://github.com/anudeepND/whitelist.git
cd whitelist/scripts
sudo ./whitelist.py
cd /opt/
sudo git clone https://github.com/anudeepND/whitelist.git
sudo vim /etc/crontab
```

2.  Once the vim editor has opened the `crontab` document, add the following line to the bottom of the file on its own line:
`0 1 * * */7 root /opt/whitelist/scripts/whitelist.py`

3. Save your edits and quit the editor. This addition to the `crontab` file will make the RasPi check for whitelist updates every Sunday.

---
### Deploying the PiHole over the network

Deploying the PiHole to the network will vary from user to user. If you want to use the PiHole on individual devices without worrying about the entire network using it, you'll need to change the DNS server for the device's network connection to the reserved IP address of the RasPi. 

For example, telling a Windows 10 computer to use the PiHole as it's DNS server can be done by going to `Control Panel > Network and Sharing Center > Change adapter settings > Ethernet > Properties > Internet Protocol Version 4 (TCP/IPv4) > Use the following DNS server addresses` and setting the "Preferred DNS server" to the reserved IP address of the RasPi.

If you would like to use the PiHole network-wide, you'll need to set the DNS server for the network to the reserved IP address of the RasPi. This is done by logging into your network's main router, going to the Internet configuration tab, and changing the primary DNS server address to that of the RasPi. After saving it, you may need to restart your router.

---
### Miscellaneous Notes
- It may be worthwhile to set a secondary Upstream DNS Provider just in case the one you selected doesn't have the DNS result that was requested. This can be done by going to the PiHole admin interface, selecting "Settings" from the left-side navigation bar, and choosing "DNS" from the top of the page. You can use the check boxes from the displayed table to activate whatever Upstream DNS Provider you want, and then click save at the bottom of the page to activate your choices. By default, I use Cloudflare are the primary provider (as selected during the configuration of the PiHole, but I also have `Google (ECS)` selected as a backup just in case.
---
