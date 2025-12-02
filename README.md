# beaconBackups
Writeup about the process and logic about making a backup system that is platform agnostic and destination irrelevant.

## Software
This system takes advantage of a couple of pre exhisting softwares to make this all happen:
- Restic Backup: This is the software that manages all the backing up and secuity of the data you want saved. It is a well consitered software suite and you can refer to their documentation for any more advanced configurations that what i have here.
- Wireguard: Wireguard will be handling the secure communication from the local file sorce to the backup location.
- AutoFS: AutoFS will handle accidental power outages and wireguard VPN connection issues for using that VPN tunnel to mount the remote drive/folder. It works by not mounting the mount point untill it is accessed and then dismounts after a set amount of time if left inactive.

Some of the temporary provisions include:
- Watchdog for restarting the computer if either the backup drive or VPN connection arent detected
- A listing in fstab for mounting the external drive indtead of using autoFS (have not been able to succesfully get the drives to mount).
- The wireguard setup instructions seem to accidentaly remove the ability for the local network to connect to access the device in any way. This to me is an accidental win, and im going to look into whats actually going on and make this a purposeful feature. (it could have to do with wireguard taking over the only port and if the local network happend to have the same subnet you may be able to ssh into the device, but this is just a theory.)

Future plans:
- Full disk encryption for the OS drive.
- Implement Backrest or something like it for a UI for restic.
- If not using backrest, make a scheduled backup system like a cron or some other way like a VM that will send the backup schedule to the device (probably not but just a thought)
- Uptime can be easily monitored by something like UptimeKuma pinging the IP of the VPN, but more comprehensive logging will be looked into.

## Hardware
This setup can work with just about any hardware you have on hand but i wanted something specific.

I wanted this to have some of the cheepest, smallest and most power efecient hardware that I could find with gigabit eathernet and a minimum of 1 USB3 port.

I landed on the [FriendlyELEC NanoPi NEO3-LTS](https://www.friendlyelec.com/index.php?route=product/product&product_id=279). It runs a bit hot, but I have had no issues with it so far.

I also needed a sata hard drive usb3 adapter with auxilary power to feed to the 3.5in HDD in the enclosure, if there isnt an aux power adapter, a 3.5in drive will not receive enough power from the USB3 port to opperate at full power. [This](https://www.amazon.com/dp/B0CYLK9QFG?ref_=ppx_hzsearch_conn_dt_b_fed_asin_title_4&th=1) is the one I have and made the 3D Model around.

As a pile of parts, this is sufficient to get started but you will also need an enclosure, a fan and power if you want to make it compact.

I have designed a 3D printable enclosure that can hold the NanoPi NEO3, the 3.5in HDD, the sata adapter, a 40mmx20mm 5v fan and a buck converter to power everything.

* [3D Model STL]()
* [Wiring Instructions]()
* Armbian install instructions
