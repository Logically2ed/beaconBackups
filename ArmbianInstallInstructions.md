## Setup:

W.I.P.
This is for an arm based system with a Debian based OS using NFS shares on the tartget machine. I am assuming that you already know how to set up wireguard on the home networks modem/router and setup an NFS share to be accessed by the beaconBox.

1. Armbian Linux setup:
	- Download the base image from the armbian website [here](https://www.armbian.com/nanopineo3/), and flash to an sd card with either belena etcher or the raspberry pi imaging tool.
	- Once the computer has boot up for the first time, you can either ssh into it or use the serial port to to communicate with it.
	- The default user name and password is root:1234
	- You will be prompted to complete the setup process.
	- Once completed, update the OS `apt-get update | apt-get upgrade`
	- Run `armbian-config`
	- go to localization and change system hostname to: backup(x).something.xyz
2. Install packages
	- Required: wireguard, autoFS, restic, net-tools
	- `apt-get install wireguard autofs restic net-tools`
	- Optional: `apt-get install btop tmux watchdog`
3. Configure Wireguard:
	-  [Reference](https://github.com/sergibarroso/wireguard-vpn-setup)
	- `mkdir /etc/wireguard/keys`
	- `chmod 700 /etc/wireguard/keys`
	- `wg genkey > /etc/wireguard/keys/private.key`
	- `cat /etc/wireguard/keys/private.key | wg pubkey > /etc/wireguard/keys/public.key`
	- `chmod 400 /etc/wireguard/keys/*.key`
	- `nano /etc/wireguard/wg0.conf`
    - Paste the following into the open text box and change everything with `<>` to the values you need:
	    - ```
	      [Interface]
			Address = <IP_AND_RANGE_FOR_THIS_MACHINE #(ex. 10.1.80.2/24)>
			PrivateKey = <CLIENT_PRIVATE_KEY>
			PreUp = sysctl -w net.ipv4.ip_forward=1
			PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o end0 -j MASQUERADE
			PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o end0 -j MASQUERADE
			PostDown = sysctl -w net.ipv4.ip_forward=0
			[Peer]
			PublicKey = <VPN_SERVER_PUBLIC_KEY>
			Endpoint = <VPN_SERVER_PUBLIC_IP>
			AllowedIPs = <IPS_YOU_WANT_THIS_COMPUTER_TO_BE_ABLE_TO_CONNECT_TO #(ex. 10.1.80.0/24, 10.1.10.5/24)>
			PersistentKeepalive = 25
	      ```
	- To keep it running indefinitely, run:
		- `systemctl enable wg-quick@wg0.service`
		- `systemctl start wg-quick@wg0.service`
		- Check status with : `sudo systemctl status wg-quick@wg0.service`
		- You should see "enabled" highlighted in green
4. Configure Autofs:
	- Navigate to `/mnt` and make a new folder i am going to call `remoteBackup`
	- To configure autofs, navigate to /etc
		- `cd /etc`
	- Edit the auto.master file
		- `nano auto.master`
	- Scroll to the bottom and add the line:
		- ```
		  /mnt/remoteBackup /etc/auto.nfs --browse --timeout=60
		  ```
	- In the /etc directory, create a new file named auto.nfs by typing: `nano auto.nfs`
	- In that new file, add the drives and network shares you want to mount using the following format. Repeat for each share you want to mount:
		- ```
		  <foldername> -fstype=nfs4,ro <STORAGE_SERVER_IP_ADDRESS>:/<PATH_TO_SHARE>
		  ```
	- Restart autofs
		- `systemctl restart autofs`
5. Plug in USB HardDrive and get the UUID
	- Mount USB Drive
		- `mkdir /mnt/USBdrive`
		- `blkid` to find the drives UUID
		- add to /etc/fstab
		- UUID= <drive_uuid> /mnt/backup ext4 defaults 0 0
		- `mount -a`
6. Configure Restic:
	- `restic init --repo /mnt/USBdrive`
	- Type in new password and follow the prompts.
7.  Configure Watchdog
	- nano /etc/watchdog.conf
	- edit the lines:
		- ```
		  log-dir = /var/log.hdd/watchdog
		  interface = wg0
		  ping = <REMOTE_WG_IP>
		  file = /mnt/backup/imhere.wtf
		  retry-timeout = 300
		  interval = 30
		  ```
8. Run restic backup
	1. restic -r /mnt/USBdrive/<Something_Repo> backup --no-scan /remoteBackup/<SHARE_NAME_TO_BACKUP>