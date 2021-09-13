
## Guide Steps

1. [Installing and configuring PiVPN](#Installing-and-configuring-PiVPN)
2. [A note on VPN connection types](#A-note-on-VPN-connection-types)
3. [Creating a user with a Full-Tunnel](#Creating-a-user-with-a-Full-Tunnel)
4. [Creating a user with a Split-Tunnel](#Creating-a-user-with-a-Split-Tunnel)
5. [Port Forwarding](#Port-Forwarding)

---
### Installing and configuring PiVPN
1. While logged in with SSH, run `curl -L https://install.pivpn.io | bash` to begin the automatic installation process. (Include everything from `sudo` to `bash` in the same command.)
2. Once the installer shows the message "This installer will transform your Raspberry Pi into an OpenVPN or WireGuard server!", the following settings should be used:
	1. **DHCP Reservation:** You'll likely want to select "Yes" here, as the PiHole should have a DHCP reservation from earlier. If not, you'll need to designated a static IP address by selecting "No"
	2.  **Local Users:** Select the user "pi". For me it's the only user, but you may have others.
	3. **Installation Mode:** This tutorial is for WireGuard. If you want to use OpenVPN, you'll have to find an older guide. Given that, select "WireGuard" from the menu.
	4. **Default WireGuard Port:** Here you designate what port you want WireGuard to communicate over.  You can leave this as the default (`51820`), or change it here. Changing it to something else can offer basic protection from script kiddies running default port scans. Other than that, there isn't a huge benefit to choosing a different value.
	5. **DNS Server Selection:** At this point the PiVPN installer should pick up on the existing installation of PiHole. If not, you may need to tell WireGuard to use the PiHole's local IP address as it's DNS server.
	6. **Public IP or DNS:** If you have a DDNS service (such as [No-IP](https://www.noip.com/)) connected to your local network, you can select "DNS Entry" from the menu to enter the domain name. If you don't use a DDNS service, you'll want to select the IP address option instead. The downside to using a direct IP is that if your ISP changes your public address (which is extremely common for residential services), you can loose connection to your VPN server and have to reconfigure it later with the newest IP address.
	7. **Server Information:** Selecting "Ok" here will allow PiVPN to generate your Server Keys
	8. **Unattended Upgrades:** Since your server will have a port open to the internet, it's possible that your device could become the target of an outside attacker using a newly-discovered exploit. Unattended Upgrades will allow your PiVPN server to install security updates without the need for you to manually intervene. **_I strongly urge you to enable this!_**
	9. **Installation Complete:** The installation is now done! Go ahead and select "Ok", and then "Yes" to reboot the PiHole device. 

---
### A note on VPN connection types
WireGuard will allow you to create profiles with different connection types, the main two being Full-Tunnel and Split-Tunnel. Below are the two types compared. You can choose which one you want to use:
 - Full-Tunnel: All traffic is sent from your device, through the VPN tunnel, and out of the network your PiHole resides in. This means all traffic from the device will have the PiVPN server's public IP address.
 - Split-Tunnel: Traffic is only sent through your network if it's attempting to reach an internal resource, or if it's a DNS request. All other traffic will come directly from your connected device, meaning your public IP address won't be masked

Full-Tunnel is a more secure choice since all traffic is encrypted and sent to your PiVPN server first, but this can result in slower speeds. Split-Tunnel is faster since it only handles DNS requests and traffic directed towards internal devices, but isn't as secure because the rest of the traffic isn't encrypted.

---
### Creating a user with a Full-Tunnel
1. Start by running the command `pivpn add`, and entering a name for the configuration file. Once you've done so, configuration files will be place in `~/configs/` and `/etc/wireguard/configs/`. If you want to make edits to your config file, I would highly recommend changing the one in `/etc/wireguard/configs/`, as that's the config file that's read during the QR code generation process. If you edit the one in `~/configs/` and then generate a QR code, it won't have any of your changes included.
2. WireGuard attempts to be as quiet as possible, meaning that it only sends and receives packets when it needs to. For this reason, clients behind a NAT or firewall might be required to keep the connection alive even when it’s not in use. To do this, you need to make a change in the server configuration file.
	1. Run the command `sudo vim /etc/wireguard/wg0.conf`
	2. Add a new line above `### begin [clientName] ###`, and insert `PersistentKeepalive = 25`
3. You're now done creating a full-tunnel profile! You can copy the config file from `/etc/wireguard/configs/` to the device you want to connect with, or if you have a camera-enabled device, you can use the command `pivpn -qr` to create a scannable QR code that contains the configuration information. **_DO NOT SHARE THE QR CODE WITH ANYONE. It contains the information needed to connect to your VPN, including the private key._**

---
### Creating a user with a Split-Tunnel
1. Start by running the command `pivpn add`, and entering a name for the configuration file. Once you've done so, configuration files will be place in `~/configs/` and `/etc/wireguard/configs/`. If you want to make edits to your config file, I would highly recommend changing the one in `/etc/wireguard/configs/`, as that's the config file that's read during the QR code generation process. If you edit the one in `~/configs/` and then generate a QR code, it won't have any of your changes included.
2. WireGuard attempts to be as quiet as possible, meaning that it only sends and receives packets when it needs to. For this reason, clients behind a NAT or firewall might be required to keep the connection alive even when it’s not in use. To do this, you need to make a change in the server configuration file.
	1. Run the command `sudo vim /etc/wireguard/wg0.conf`
	2. Add a new line above `### begin [clientName] ###`, and insert `PersistentKeepalive = 25`
	3. You'll also need to add the following lines to your server config file (`sudo vim /etc/wireguard/wg0.conf` again) underneath `[Interface]` at the top of the file:
	```
	PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE  
	PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
	```
3. By default the configuration files created by `pivpn add` are set for Full-Tunnel. To change this, we'll need to edit the config file that was created earlier
	1. Run the command `sudo vim /etc/wireguard/configs/[configName].conf`
	2. Locate the line that starts with `AllowedIPs` and delete everything after the equals (`=`) sign, replacing it with the local IP range. Typically this will look something like `AllowedIPs = 192.168.x.0/24` for a smaller private network
	3. Locate the line that starts with `DNS` at the top of the file and delete everything after the equals (`=`) sign, replacing it with the local  IP address of your PiHole server. For me, this would look something like `DNS = 192.168.2.254`.
	4. Save your changes and quit the editor
4. You're now done creating a split-tunnel profile! You can copy the config file from `/etc/wireguard/configs/` to the device you want to connect with, or if you have a camera-enabled device, you can use the command `pivpn -qr` to create a scannable QR code that contains the configuration information. **_DO NOT SHARE THE QR CODE WITH ANYONE. It contains the information needed to connect to your VPN, including the private key._**

---
### Port Forwarding
This step is going to be completely different for each user depending on the brand of router being used in the network. While I can't guide you through the process of opening a port, the requirements for the port forwarding are below:
- **Service Name:** User's choice
- **Protocol:** UDP
- **External Port Range:** Whatever port was chosen [here](#Installing-and-configuring-PiVPN) on step 4
- **Internal Port Range:** Same as external port range
- **Internal IP Address:** The local IP address of the PiHole device

Once port forwarding has been completed, and the previously-created client configuration file has been loaded onto your connecting device, you can turn the VPN on and you should begin to see remote traffic flowing into your PiHole admin interface! 
