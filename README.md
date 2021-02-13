# SMA Home Manager printer

Many tanks to mitchese for this work. I did a copy to explain the installation process in more detail.

This simple program prints the values of the [SMA Home Manager](https://www.sma.de/en/products/monitoring-control/sunny-home-manager-20.html)

I wrote this as an initial test before implementing a fake ET340 in a Victron system. Use this at your own risk, I have
no association with Victron but am providing this for anyone who already has an SMA Home Manager (and no room left in their fuse box for a further meter)


# Prepare

In order to get your arm file for Victron Multiplus II GX you need to compile the file. I have done this on a Windows machine. Follow these steps in order to get your armv7 file.

1. Download and install go environment
2. Check your env GOPATH=  (go env) and take note of the path. Set GOOS=linux GOARCH=arm
3. Download the files form mitchese
4. In the file is a reference to https://github.com/dmichael/go-multicast 
4.1 Download this repo too and copy it in your GOPATH /src folder.  --> C:\Users\"yourusername"\go\src\go-multicast-master
5. You now have to adjust the import folder for go-multicast-master. In my case the GOPATH=C:\Users\"yourusername"\go and I had to modify C:\Users\"yourusername"\go\src\go-multicast-master\main.go file. In import section (line 13) i corrected the path as follow: "go-multicast-master/multicast".
6. You now modify the sma_home_manager_printer part the same way. In my case again 	"go-multicast-master/multicast".
7. Finally you can build your file.Open a command promt and move to the downloaded and previously modified file sma_home_manager_printer.go. Compile it with: go build -o sma_home_manager_printer.armv7

Now you need to prepare your Multiplus:

1. Switch on the Multiplus and connect the ethernet port to a router or a network switch. I had a switch only.
2. the IP address wil be printed on the screen of the Multiplus. Remember it runs his own DHCP Server. 
3. Open a browser window and insert the IP Address of the Multiplus.
4. You will land on the mainpage. Use your keybaord cursors to navigate. Settings --> General --> Set root passwor. Do so and choos a strong password. Add it to your password manager. If you don't have it, its time to organize one.
5. Set the Access Level to User and installer, the password is ZZZ
6. Highlight Access Level (don't open the select page!)
7. Press and hold the right curser of your keyboard until you see the Access Level change to Superuser.
8. Go to Settings → General → Set root password. And create a root password. Note that, for firmware version v2.00 and later, the root password will be reset by a firmware update. The reason is that the passwd file is on the rootfs, which is fully replaced by an update.
9. Finally you activate SSH.
10. Install first, or if you have it already, open putty and connect to your Multiplus through SSH. You also can use MobaXterm is you like to have more copy paste style.

https://www.victronenergy.com/live/ccgx:root_access


## How it works

The SMA Home Manager publishes every second a UDP message which includes all the values, both instantaneous usage as well as counters.

In order to 'fake' an ET340 in the Victron system, this program will subscribe to the updates of the meter and inject these into dbus.

This program subscribes and decodes the important values, which are used to test if the connection works. It prints these out for manual verification

To run, just run in a terminal

## Testing on Victron Venus OS

You can use this program to verify that your Venus OS can connect to an SMA meter, instead of using the ET340/EM24 meters.

I have tested this on the Victron Venus GX, but the same should work for any of the Victron GX devices. In the menu:

* Activate "Superuser" under Settings -> General (see https://www.victronenergy.com/live/ccgx:root_access)
* Set an SSH Password and enable SSH on LAN
* Download the `sma_home_manager_printer.armv7l` to your computer
* Use WinSCP or other scp program to copy `sma_home_manager_printer.armv7l` to your Venus
* Use Putty (or similar) to ssh to the Venus GX. Run with `./sma_home_manager_printer.armv7l`

Output should resemble the following:
```
2020/05/27 20:47:45 -----------------------------------------------------
2020/05/27 20:47:45 Received datagram from meter
2020/05/27 20:47:45 Uid:  253910
2020/05/27 20:47:45 Serial:  3004906401
2020/05/27 20:47:45 Total W:  484
2020/05/27 20:47:45 Total Buy kWh:   6675.477900000001
2020/05/27 20:47:45 Total Sell kWh:  3200.4507000000003
2020/05/27 20:47:46 +-----+-----------+-----------+-----------+
2020/05/27 20:47:45 |value|   L1      |     L2    |   L3      |
2020/05/27 20:47:45 +-----+-----------+-----------+-----------+
2020/05/27 20:47:45 |  V  |   231.18  |   230.75  |   231.35  |
2020/05/27 20:47:45 |  A  |    -0.11  |     0.52  |     1.68  |
2020/05/27 20:47:45 |  W  |   -24.90  |   120.90  |   388.10  |
2020/05/27 20:47:45 | kWh |  3084.11  |  1213.50  |  3002.64  |
2020/05/27 20:47:45 | kWh |  1309.15  |  1599.17  |   916.91  |
2020/05/27 20:47:45 +-----+-----------+-----------+-----------+
2020/05/27 20:47:46 -------------------------------------------
2020/05/27 20:47:46 Received datagram from meter
2020/05/27 20:47:46 Uid:  253910
2020/05/27 20:47:46 Serial:  3004906401
2020/05/27 20:47:46 Total W:  490.6
2020/05/27 20:47:46 Total Buy kWh:   6675.478
2020/05/27 20:47:46 Total Sell kWh:  3200.4507000000003
2020/05/27 20:47:46 +-----+-----------+-----------+-----------+
2020/05/27 20:47:46 |value|   L1      |     L2    |   L3      |
2020/05/27 20:47:46 +-----+-----------+-----------+-----------+
2020/05/27 20:47:46 |  V  |   231.57  |   230.76  |   231.03  |
2020/05/27 20:47:46 |  A  |    -0.11  |     0.55  |     1.68  |
2020/05/27 20:47:46 |  W  |   -24.80  |   127.60  |   387.80  |
2020/05/27 20:47:46 | kWh |  3084.11  |  1213.50  |  3002.64  |
2020/05/27 20:47:46 | kWh |  1309.15  |  1599.17  |   916.91  |
2020/05/27 20:47:46 +-----+-----------+-----------+-----------+
```

You can verify the Serial number with the number shown on your SMA inverter under Device Configuration. The rest of
the values should make sense.

## Problems

If this program begins outputting data as above, but stops within a few minutes, ensure that "IGMP Snooping" is enabled
on your network switches/routers (whatever's between this and the home manager). This is necessary for multicast.
I did not have this enabled on my switch, _somehow_ the SMA inverters did not have an issue (they must have constantly
re-connected). This program should run forever until stopped.

