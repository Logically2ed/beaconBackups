## Setup:

W.I.P.

Armbian Linux setup:
1. Initialize OS
	1. un:root - pass:random
	2. un:backup# - pass:random
	3. Name : Backup#
	4. run updates
	5. run armbian-config
		1. go to localization and change the hostname to: Hostname = backup(x).something.xyz
2. Install and Configure Wireguard
	1. install wireguard
	2. [Reference](https://github.com/sergibarroso/wireguard-vpn-setup)
	3. `mkdir /etc/wireguard/keys`
	4. `chmod 700 /etc/wireguard/keys`
	5. `wg genkey > /etc/wireguard/keys/private.key`
	6. `cat /etc/wireguard/keys/private.key | wg pubkey > /etc/wireguard/keys/public.key`
	7. `chmod 400 /etc/wireguard/keys/*.key`
	8. `nano /etc/wireguard/wg0.conf`
    9. Paste the following into the open text box and change everything with `<>` to the values you need.
> [Interface]
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

    10. to keep it running indefinitely:
		  1. `systemctl enable wg-quick@wg0.service`
		  2. `systemctl start wg-quick@wg0.service`
		  3. check status with : `sudo systemctl status wg-quick@wg0.service`
3. Install and Configure Autofs and fstab
	1. install Autofs
	2. Edit auto.master file with all the mounted nfs shares and usb drives
	3. Mount NFS drive
    1. navigate to the root of the drive
       `cd /`
		2. mkdir <serverName>
			1. mkdir <shareName>
		2. Make an auto.*** file for each share to be backed up and add these to the respective file.
			1. `sudo nano auto.shared`
				1. shared  -fstype=nfs4  <IP>:/<path-to-share>
	5. sudo systemctl restart autofs
	6. Mount USB Drive
		1. `mkdir /mnt/backup`
		2. `blkid` to find the drives UUID
		3. add to /etc/fstab
		4. UUID= <drive uuid /mnt/backup ext4 defaults 0 0
		5. `mount -a`
4. Install Restic
	1. apt install restic
	2. restic init --repo /mnt/something-repo
	3. type in new password
5.  Install watchdog
	1. nano /etc/watchdog.conf
	2. edit the lines:
		1. log-dir = /var/log.hdd/watchdog
		2. interface = wg0
		3. ping = <REMOTE_WG_IP>
		4. file = /mnt/backup/imhere.wtf
		5. retry-timeout = 300
		6. interval = 30
6. Run restic backup and setup scheduled backups
	1. restic -r /mnt/backup/something-repo backup --no-scan /<serverName>/<shareName>/<folderToBackup>
